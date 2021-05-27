---
layout: post
title: Kubernetes Config
summary: Kubernetes
author: devhtak
date: '2021-05-23 21:41:00 +0900'
category: Container
---

#### 컨테이너 환경 변수 전달 방법

- 도커 컨테이너의 환경 설정
  - 쿠버네티스의 환경 변수는 YAML 파일이나 다른 리소스로 전달
  - 하드 코딩된 환경 변수는 여러 환경에 데이터를 정의하거나 유지, 관리가 어려움
  - ConfigMap은 외부에 컨테이너 설정을 저장할 수 있음
    - YAML에 하드 코딩된 환경변수는 모든 파일을 수정해야 하는 불편함이 있지만 ConfigMap을 활용하면 빠르게 적용할 수 있다.

- 도커 컨테이너의 환경 설정
  - 일반적인 Key-Value 형태
    ```
    env:
    - name: DEMO_GREETING
      value: "HELLO FROM THE ENV"
    ```
    
  - ConfigMap
    ```
    env:
    - name: DEMO_GREETING
      valueFrom:
        configMapKeyRef: confimap-name
    ```
    
  - Secrets
    ```
    env:
    - name: DEMO_GREETING
      valueFrom:
        secretKeyRef: secret-name
    ```

- 환경 변수를 포드에 저장하는 방법
  - node-env.yaml
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: envar-demo
      labels:
        purpose: demonstrate-envars
    spec:
      containers:
      - name: envar-demo-container
        image: gcr.io/google-samples/node-hello:1.0
        env:
        - name: DEMO_GREETING
          value: "Hello From The Environment"
        - name: DEMO_FAREWELL
          value: "Such a sweet sorrow"
    ```
    ```
    $ kubectl exec -it envar-demo -- /bin/bash
    root@envar-demo:#/ printenv
    // ...
    DEMO_FAREWELL=Such a sweet sorrow
    // ...
    DEMO_GREETING=Hello From The Environment
    // ...
    ```
    
#### ConfigMap을 활용한 환경 변수 설정

- 환경 변수를 ConfigMap에 저장하는 방법
  - kubectl 명령어를 사용하여 바로 저장할 수 있음
    ```
    $ kubectl create configmap <map-name> <data-source>
    ```
  - test 파일을 생성하고 그 파일을 환경 변수로 저장
    - from-file을 여러개 주어 여러 값을 사용할 수 있다.
      ```
      $ echo -n 1234 > test
      $ kubectl create configmap map-name --from-file=test
      configmap/map-name created
      ```
  - 저장한 내용 확인
    - -o yaml로 yaml 파일 형식으로 확인할 수 있다.
      ```
      $ kubectl get configmap map-name -o yaml
      apiVersion: v1
      data:
        test:
          1234
      kind: ConfigMap
      [중략]
      ```

- yaml을 사용하여 환경변수를 ConfigMap에 저장하는 방법
  - POD 생성할 때 configmap 설정
    ```
    // ...
    env:
    - name: SPECIAL_LEVEL_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.how
    // ...
    ```
  - configmap 생성
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata
      # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
      name: special-config
      # Specify the key associated with the value
      namepsace: default
    data:
      special.how: very
    ---
    apiVersion: v1
    kind: COnfigMap
    metadata:
      name: env-config
      namespace: default
    data:
      log_level: INFO
    ```

#### ConfigMap을 활용한 Mounting

- Add ConfigMap data to a Volume
- 예제
  - ConfigMap 생성
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: special-config
      namespace: default
    data:
      SPECIAL_LEVEL: very
      SPECIAL_TYPE: charm
    ```
  - POD 생성
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: volumes-configmap
    spec:
      containers:
      - name: test-container
        image: k8s.gcr.io/busybox
        command: ["/bin/sh", "-c", "ls /etc/config"]
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          # Provide the name of the ConfigMap containing the files you want to add to the container
          name: special-config
      restartPolicy: Never
    ```
    - /etc/config에 special-config를 마운트하는 것
    - special-config에 설정한 key를 이름으로 한 파일형태로 value가 내용으로 저장되어 있다.
      ```
      $ kubectl exec -it volumes-configmap -- bash
      $ cd /etc/config
      $ ls
      SPECIAL_LEVEL  SPECIAL_TYPE
      ```

#### Secret을 활용한 환경변수 설정

- secret은 비밀번호, OAuth 토큰, SSH 키와 같은 민감한 정보를 encoding하여 저장한다.
- base64로 encoding되어 있기 때문에 디코딩도 가능하기 때문에 공격자에게 혼란을 줄 뿐 보안이 완벽하진 않다.
- 환경 변수를 Secret에 저장하는 방법
  - kubectl 명령어로 secret 설정하는 방법
    ```
    $ echo -n 'admin' > ./username
    $ echo -n '1q2w3e4r5t' > ./password
    $ kubectl create secret generic db-user-pass --from-file=./username --from-file=./password
    secret/db-user-pass created
    $ kubectl get secret db-user-pass -o yaml
    apiVersion: v1
    data:
      password: MXEydzNlNHI1dA==
      username: YWRtaW4=
    kind: Secret
    metadata:
      creationTimestamp: "2021-05-23T11:49:27Z"
      name: db-user-pass
      namespace: default
      resourceVersion: "89868"
      uid: dd38add1-ef0a-4087-a1f8-fcdd7efe9411
    type: Opaque
    ```
    - echo -n 명령어를 실행하면 데이터가 개행문자(엔터)와 함께 출력
    - 마지막 명령어를 실행하면 db-user-pass 유저가 생성
    - secret에는 DB의 user와 password와 같이 민감한 파일들이 base64 인코딩돼서 저장
    - 해당 값이 안전한 것은 아니지만 공격자에게 혼란을 줄 수 있다.
  - Pod에 secret으로 key-value 설정
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: envar-secret
      labels:
        purpose: demonstrate-envars
    spec:
      containers:
      - name: envar-demo-container
        image: gcr.io/google-samples/node-hello:1.0
        env:
        - name: user
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: username
        - name: pass
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: password
    ```

- 수동으로 데이터를 만들어야 하는 경우에는 base64 인코딩을 수동으로 해주어야 한다.
  ```
  $ echo admin | base64
  YWRtaW4K
  $ echo YWRtaW4K | base64 --decode
  admin
  ```
  
#### 초기 명령어 및 아규먼트 전달과 실행

- 초기 실행 시 명령어와 아규먼트를 전달
  - Pod을 생성할 때 spec.container.command와 args에 실행하기 원하는 인자를 전달하면 컨테이너가 부팅된 뒤 실행
  - pod-command-args.yaml
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: command-demo
      labels:
        purpose: demonstrate-command
    spec:
      containers:
      - name: command-demo-container
        image: debian
        command: ["printenv"]
        args: ["HOSTNAME", "KUBERNETES_PORT"]
      restartPolicy: OnnFailure
    ```

- 환경 변수를 활용하여 출력할 때는 $를 사용하여 명령 내용 변경 가능
  ```
  # ...
  env:
  - name: MESSAGE
    value: "hello world"
  command: ["/bin/echo"]
  args: ["$(MESSAGE)"]
  #...
  ```

#### 하나의 포드에 멀티 컨테이너

- 하나의 포드에 다수의 컨테이너를 사용
  - 하나의 포드를 사용하는 경우 같은 네트워크 인터페이스와 IPC, 볼륨등을 공유
  - 이 포드는 효율적으로 통신하여 데이터의 지역성을 보장하고 여러 개의 응용프로그램이 결합된 형태로 하나의 포드를 구성할 수 있음
  - 모니터링 서비스 등에서 사용한다.

- 예제
  - pod-multi-container.yaml
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: two-containers
    spec:
      restartPolicy: Never
      volumes:
      - name: shared-data
        emptyDir: {}
      containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
      - name: debian-container
        image: debian
        volumeMounts:
        - name: shared-data
          mount-path: /pod-data
        command: ["/bin/sh"]
        args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
    ```
  - debian 컨테이너에서 실행한 /pod-data/index.html이 제대로 mount되어 있는지 확인
    ```
    $ kubectl exec -it two-containers -- cat /usr/share/nginx/html/index.html
    Defaulted container "nginx-container" out of: nginx-container, debian-container
    Hello from the debian container
    ```

#### init container

- init container 특징
  - 포드 컨테이너 실행 전에 초기화 역할을 하는 컨테이너
  - 완전히 초기화가 진행된 다음에야 주 컨테이너를 실행
  - init 컨테이너가 실패하면, 성공할 때까지 포드를 반복해서 실행
    - restartPolicy: Never를 주면 재시작하지 않는다.

- 예제
  - init container 실행 후 실행되는 container
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ["sh", "-c", "echo The app is running! && sleep 3600"]
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
    ```
    - 해당 yaml은 mydb와 myservice가 탐지될 때까지 init 컨테이너가 멈추지 않고 돌아간다.
  - init process를 끝낼 수 있는 종결자!
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: myservice
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: mydb
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9377
    ```
    - status가 Init:0/2에서 Init:1/2, Init:2/2, PodInitializing, Running 순으로 변경된다.

#### 시스템 리소스 요구사항과 제한 설정

- 컨테이너에서 리소스 요구사항
  - CPU와 메모리는 집합적으로 컴퓨팅 리소스 또는 리소스로 부른다.
  - CPU 및 메모리는 각각 자원 유형을 지니며 자원 유형에는 기본 단위를 사용
  - 리소스 요청 설정 방법
    ```
    > spec.containers[].resources.requests.cpu
    > spec.containers[].resources.requests.memory
    ```
  - 리소스 제한 설정 방법
    ```
    > spec.containers[].resources.limits.cpu
    > spec.containers[].resources.limits.memory
    ```
  - CPU는 코어 단위로 지저오디며 메모리는 바이트 단위로 지정
  
    |자원 유형|단위|
    |---|---|
    |CPU|m(millicpu),|
    |Memory|--- Ti, Gi, Mi, Ki, T, G, M, K|
    
    - CPU 0.5가 있는 컨테이너는 CPU 1개를 요구하는 절반의 CPU
    - CPU 0.1은 100m 와 동일한 기능
    - K, M, G 의 단위는 1000씩 증가
    - Ki, Mi, Gi의 단위는 1024씩 증가

  - 환경에 따른 CPU의 의미
    - 1 AWS vCPU
    - 1 GCP Core
    - 1 Azure vCore
    - 1 IBM vCPU
    - 1 하이퍼 스레딩 기능이 있는 베어 메탈 인텔 프로세서의 하이퍼 스레드

- 예제
  - pod-resource.yaml
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend
    spec:
      containers:
      - name: db
        image: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "126Mi"
            cpu: "500m"
      - name: wordpress
        image: wordpress
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "126Mi"
            cpu: "500m"
    ```
    - 각각의 컨테이너마다 리소스를 지정할 수 있다.

#### 컨테이너 리소스 제한 및 요구사항 설정

- limitRagnes
  - https://kubernetes.io/docs/concepts/policy/limit-range/
  - 네임 스페이스에서 포드 또는 컨테이너별로 리소스를 제한하는 정책

- 리미트 레인지의 기능
  - 네임스페이스에서 포드나 컨테이너당 최소 및 최대 컴퓨팅 리소스 사용량 제한
  - 네임스페이스에서 PersistentVolumeClaim 당 최소, 최대 스토리지 사용량 제한
  - 네임스페이스에서 리소스에 대한 요청과 제한 사이의 비율 적용
  - 네임스페이스에서 컴퓨팅 리소스에 대한 디폴트 requests/limit을 설정하고 런타임중에 컨테이너에 자동으로 입력

- LimitRange 적용 방법
  - Apiserver 옵션에 --enable-admission-plugins=LimitRange를 설정
    ```
    $ sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
    # ...
    spec:
      containers:
      - command:
        - --enable-admission-plugins=NodeRestriction, LimitRange
    # ...
    ```
  - cloud에서는 Apiserver에 설정으로 할 수 있는지 확인해야 하며 vendor마다 하는 방법이 다르다.

- 예제
  - 각 컨테이너에 설정
    ```
    apiVersion: v1
    type: LimitRange
    metadata:
      name: limit-mem-cpu-per-container
    spec:
      limits:
      - max:
          cpu: "800m"
          memory: "1Gi"
        min:
          cpu: "100m"
          memory: "99Mi"
        default:
          cpu: "700m"
          memory: "900Mi"
        defaultRequest:
          cpu: "110m"
          memory: "111Mi"
        type: Container
    ```
    - type이 Container이면 각 컨테이너에 설정하는 것
    - 제한하기 원하는 Namespace에 limitRange 리소스를 생성
    - 리소스 조회
      ```
      $ kubectl describe limitrange -n namespace_name
      ```

  - 포드 수준의 리소스 제한
    ```
    apiVersion: v1
    type: LimitRange
    metadata:
      name: limit-mem-cpu-per-pod
    spec:
      limits:
      - max:
          cpu: "800m"
          memory: "1Gi"
        type: Pod
    ```
    - type이 Pod이면 각 파드에 설정하는 것
  - 스토리지 리소스 제한
    ```
    apiVersion: v1
    type: LimitRange
    metadata:
      name: storage-limits
    spec:
      limits:
      - type: PersistentVolumeClaim
        max:
          storage: 2Gi
        min:
          storage: 1Gi
    ```
    - type이 PVC이면 스토리지에 설정하는 것

#### 네임스페이스 별 리소스 총량 제한 방법: ResourceQuata

- https://kubernetes.io/docs/tasks/administrater-cluster/manager-resources/quota-memory-cpu-namespace
- 네임스페이스별 리소스 제한
  - 제한하기 원하는 네임스페이스에 ResourceQuata 리소스 생성
  - 모든 컨테이너에는 CPU, 메모리에 대한 최소 요구사항 및 제한 설정 필요

- 예제
  ```
  apiVersion: v1
  kind: ResourceQuata
  metadata:
    name: mem-cpu-demo
  spec:
    hard:
      requests.cpu: '1'
      requests.memory: 1Gi
      limits.cpu: "2"
      limits.memory: 2Gi
  ```
  - 네임스페이스 내의 모든 컨테이너의 합이 설정만큼 넘어서는 안된다.
  - 리소스 조회
    ```
    $ kubectl describe resourcequota -n namespace
    ```

#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터
