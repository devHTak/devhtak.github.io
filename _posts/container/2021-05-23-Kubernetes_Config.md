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
        test: |
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

#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터
