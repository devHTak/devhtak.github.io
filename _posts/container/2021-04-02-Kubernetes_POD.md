---
layout: post
title: Kubernetes POD
summary: Kubernetes
author: devhtak
date: '2021-04-02 21:41:00 +0900'
category: Container
---

#### POD

- 쿠버네티스가 관리하는 가장 작은 컴퓨팅 단위로 하나 이상의 컨테이너 그룹이다. 
- 컨테이너의 공동 배포된 그룹이며 쿠버네티스의 기본 빌딩 블록을 대표
- 쿠버네티스는 컨테이너를 개별적으로 배포하지 않고 컨테이너의 포드를 항상 배포하고 운영
- 일반적으로 포드는 단일 컨테이너만 포함하지만 다수의 컨테이너를 포함할 수 있음
- 포드는 다수의 노드에 생성되지 않고 단일 노드에서만 실행
- 여러 프로세스를 실행하기 위해서는 컨테이너 당 단일 프로세스가 적합
- 다수의 프로세스를 제어하려면? -> 다수의 컨테이너를 다룰 수 있는 그룹 필요

- 포드 관리의 장점
  - 포드는 밀접하게 연관된 프로세스를 함께 실행하고 마치 하나의 환경에서 동작하는 것처럼 보임
  - 그러나 동일한 환경을 제공하면서 다소 격리된 상태로 유지

- 동일한 포드의 컨테이너 사이의 부분 격리
  - 포드의 모든 컨테이너는 동일한 네트워크 및 UTS 네임스페이스에서 실행
  - 같은 호스트 이름 및 네트워크 인터페이스를 공유 (포드 충돌 가능성 있음)
  - 포드의 모든 컨테이너는 동일한 IPC 네임스페이스 아래에서 실행되며 IPC를 통해 통신 가능
    - 최신 쿠버네티스 및 도커 버전에는 동일한 PID 네임스페이스를 공유할 수 있지만 이 기능은 기본 활성화돼있지 않다.

- 플랫 인터 포드 네트워크 구조
  - 쿠버네티스 클러스터의 모든 포드는 공유된 단일 플랫, 네트워크 주소 공간에 위치
  - 포드 사이에는 NAT 게이트웨이가 존재하지 않는다.

- 컨테이너를 포드 전체에 적절하게 구성하는 방법
  - 다수의 포드로 멀티티어 애플리케이션 분할
  - 각각 스케일링이 가능한 포드로 분할
  - ex) FE Process와 BE Process를 같은 포드에 구성하기 보다는 각각의 포드로 구성하는 것이 좋다.
  - 밀접한 프로세스가 아닌 경우에는 한 포드에 여러 컨테이너를 사용하지 않는다.

- YAML로 POD Descriptor 만들기
  - kubectl 실행 명령으로 간단한 리소스 작성 방법도 가능하지만 일부 항목에 대해서만 가능하며 용의하지 않다.
  - 모든 쿠버네티스 객체를 YAML로 정의하면 버전 제어 시스템에 저장 가능
  - 모든 API에 대한 내용: https://kubernetes.io/docs/reference 참고
  
- POD 정의 구성 요소
  - apiVersion: 쿠버네티스 API의 버전을 가리킴
  - kind: 어떤 리소스 유형인지 결정(포드 레플리카컨트롤러, 서비스 등)
  - 메타데이터: 포드와 관련된 이름, 네임스페이스, 레이블, 그 밖의 정보 존재
  - 스펙: 컨테이너, 볼륨등의 정보
  - 상태: 포드의 상태, 각 컨테이너의 설명 및 상태, 포드 내부의 IP 및 그 밖의 기본 정보 등

#### POD Descriptor 작성

- yaml 파일 작성

  ```
  server1@server1-VirtualBox:~/yaml$ vi go-test.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: go-test 
  spec:
    containers:
    - name: go-test
      image: devtak/test:v1.0
      ports:
      - containerPort: 8080
  ```

- yaml 파일로 deploy 생성하기

  ```
  server1@server1-VirtualBox:~/yaml$ kubectl create -f go-test.yaml
  server1@server1-VirtualBox:~/yaml$ kubectl get pod -o yaml
  apiVersion: v1
  items:
  - apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: "2021-04-02T06:41:26Z"
      managedFields:
      - apiVersion: v1
        fieldsType: FieldsV1
        fieldsV1:
          f:spec:
            f:containers:
              k:{"name":"go-test"}:
                .: {}
                f:image: {}
                f:imagePullPolicy: {}
                f:name: {}
                f:ports:
                  .: {}
                  k:{"containerPort":8080,"protocol":"TCP"}:
                    .: {}
                    f:containerPort: {}
                    f:protocol: {}
                f:resources: {}
                f:terminationMessagePath: {}
                f:terminationMessagePolicy: {}
            f:dnsPolicy: {}
            f:enableServiceLinks: {}
            f:restartPolicy: {}
            f:schedulerName: {}
            f:securityContext: {}
            f:terminationGracePeriodSeconds: {}
        manager: kubectl-create
        operation: Update
        time: "2021-04-02T06:41:26Z"
      - apiVersion: v1
        fieldsType: FieldsV1
        fieldsV1:
          f:status:
            f:conditions:
              k:{"type":"ContainersReady"}:
                .: {}
                f:lastProbeTime: {}
                f:lastTransitionTime: {}
                f:status: {}
                f:type: {}
              k:{"type":"Initialized"}:
                .: {}
                f:lastProbeTime: {}
                f:lastTransitionTime: {}
                f:status: {}
                f:type: {}
              k:{"type":"Ready"}:
                .: {}
                f:lastProbeTime: {}
                f:lastTransitionTime: {}
                f:status: {}
                f:type: {}
            f:containerStatuses: {}
            f:hostIP: {}
            f:phase: {}
            f:podIP: {}
            f:podIPs:
              .: {}
              k:{"ip":"172.17.0.2"}:
                .: {}
                f:ip: {}
            f:startTime: {}
        manager: kubelet
        operation: Update
        time: "2021-04-02T06:41:29Z"
      name: go-test
      namespace: default
      resourceVersion: "12002"
      uid: 463e3c63-be3b-4563-805d-e158bacac1e8
    spec:
      containers:
      - image: devtak/test:v1.0
        imagePullPolicy: IfNotPresent
        name: go-test
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: default-token-8p5fp
          readOnly: true
      dnsPolicy: ClusterFirst
      enableServiceLinks: true
      nodeName: minikube
      preemptionPolicy: PreemptLowerPriority
      priority: 0
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: default
      serviceAccountName: default
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 300
      - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 300
      volumes:
      - name: default-token-8p5fp
        secret:
          defaultMode: 420
          secretName: default-token-8p5fp
    status:
      conditions:
      - lastProbeTime: null
        lastTransitionTime: "2021-04-02T06:41:26Z"
        status: "True"
        type: Initialized
      - lastProbeTime: null
        lastTransitionTime: "2021-04-02T06:41:29Z"
        status: "True"
        type: Ready
      - lastProbeTime: null
        lastTransitionTime: "2021-04-02T06:41:29Z"
        status: "True"
        type: ContainersReady
      - lastProbeTime: null
        lastTransitionTime: "2021-04-02T06:41:26Z"
        status: "True"
        type: PodScheduled
      containerStatuses:
      - containerID: docker://153a1cc21662b7620cc32fcb4638d607c24614c28d94280c321bcfe90ba49159
        image: devtak/test:v1.0
        imageID: docker-pullable://devtak/test@sha256:88b8e9599f84da43bfe13805bcce93e474a6b79c5564d535faa1b4b54eec6d99
        lastState: {}
        name: go-test
        ready: true
        restartCount: 0
        started: true
        state:
          running:
            startedAt: "2021-04-02T06:41:28Z"
      hostIP: 192.168.49.2
      phase: Running
      podIP: 172.17.0.2
      podIPs:
      - ip: 172.17.0.2
      qosClass: BestEffort
      startTime: "2021-04-02T06:41:26Z"
  kind: List
  metadata:
    resourceVersion: ""
    selfLink: ""
  ```
  
- 컨테이너에서 호스트로 포트 포워딩
  - 디버깅 혹은 다른 이유로 서비스를 거치지 않고 특정 포드와 통신하고 싶을 때 사용
  - kubectl port-forward 명령 수행
  - 컨테이너 8081 포트를 pod의 8080 포트로 전달
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl port-forward go-test 8081:8080
  Forwarding from 127.0.0.1:8081 -> 8080
  Forwarding from [::1]:8081 -> 8080
  
  http://127.0.0.1:8081/hello 연결 확인
  ``` 
  
- 삭제도 가능하다.
  ```
  server1@server1-VirtualBox:~/yaml$ kyubectl delete -f go-test.yaml
  server1@server1-VirtualBox:~/yaml$ kyubectl delete pod go-test
  ```

- 로그 확인도 가능하다.
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl logs go-test
  ```

- 포드에 주석 추가하기
  - 각 포드나 API 객체 설명이 추가
  - 클러스터를 사용하는 모든 사람이 각 객체의 정보를 빠르게 확인 가능
  - 예를 들어 객체를 만든 사람의 이름을 지정
  - 공동 작업 가능
  - 총 256KB까지 포함 가능
    ```  
    server1@server1-VirtualBox:~/yaml$ kubectl annotate pod http-go key="test1234"
    pod/http-go annotated
    server1@server1-VirtualBox:~/yaml$ kubectl get pod http-go -o yaml
    ```
    
- 포드 삭제
  - 만든 포드 조회
  - 포드 삭제
    ```
    server1@server1-VirtualBox:~/yaml$ kubectl delete pod <포드 이름>
    ```
  - 전체 포드 삭제
    ```
    server1@server1-VirtualBox:~/yaml$ kubectl delete pod --all
    ```

#### 라이브니스, 레디네스, 스타트업 프로브 구성

- Liveness Probe
  - 컨테이너 살았는지 판단하고 다시 시작하는 기능
  - 컨테이너의 상태를 스스로 판단하여 교착 상태에 빠진 컨테이너를 재시작
  - 버그가 생겨도 높은 가용성을 보입

- Readiness Probe
  - 포드가 준비된 상태에 있는지 확인하고 정상 서비스를 시작하는 기능
  - 포드가 적절하게 준비되지 않은 경우 로드밸런싱을 하지 않음

- Startup Probe
  - 애플리케이션의 시작 시기 확인하여 가용성을 높이는 기능
  - Liveness, Readiness의 기능 비활성화

- Liveness Command 설정 - 파일 존재 여부 확인
  - 리눅스 환경에서 커맨드 실행 성공 시 0 (컨테이너 유지)
  - 실패하면 그 외 값 출력 (컨테이너 재시작)
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      test: liveness
    names: liveness-exec
  spec:
    containers:
    - name: liveness
      image: k8c.gcr.io/busybox
      args:
      - /bin/sh
      - -c
      - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      livenessProve:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
  ```
    - initialDelaySeconds: 5 -> POD이 실행되고 5초 뒤부터 확인
    - periodSeconds: 5 -> 5초마다 확인
    - spec.containers.args 에서 /tmp/healthy를 생성하고 삭제했다.
    - livenessProve는 /tmp/healthy를 확인한다. 즉, 30초 뒤에 삭제하기 때문에 30초 뒤에 컨테이너가 죽었다고 판단하고 다시 실행한다.
    - 아래 명령어로 확인할 수 있다.
      ```
      $ kubectl describe pod liveness-exec
      ```
  
- Liveness 웹 설정 - http 요청 확인
  - Response Code가 200 이상, 400 미만: 컨테이너 유지
  - Response Code가 그 외(500 server error): 컨테이너 재시작
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      test: liveness
    names: liveness-http
  spec:
    containers:
    - name: liveness
      image: k8c.gcr.io/busybox
      args:
      - /server
      livenessProve:
        httpGet:
          path: /healthz
          port: 8080
          httpHeaders:
          - name: Custom-Header
            value: Awesome
        initialDelaySeconds: 3
        periodSeconds: 3
  ```

- TCP 설정
  - TCP 연결로 확인
  - Readiness TCP 설정
    - 준비 프로브는 8080포트를 검사
    - 5초 후부터 검사 시작
    - 검사 주기는 10초 -> 서비스를 시작해도 된다.

  - Liveness TCP 설정
    - 활성화 프로브는 8080포트를 검사
    - 15초 후부터 검사 시작
    - 검사 주기는 20초 -> 컨테이너를 재시작하지 않아도 된다.
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    names: goproxy
    labels:
      app: goproxy
  spec:
    containers:
    - name: goproxy
      image: k8c.gcr.io/goproxy:0.1
      ports:
      - containerPort: 8080
      readinessProve:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        tcpSocket:
          port:8080
        initialDelaySeconds: 3
        periodSeconds: 3
  ```  

- Startup Probe
  - 시작할 때까지 검사를 수행
  - http 요청을 통해 검사
  - 30번을 검사하며 10초 간격으로 수행
  - 300(30 * 10)초 후에도 포드가 정상 동작하지 않는 경우 종료 -> 300초 동안 포드가 정상 실행되는 시간을 벌어준다.
  
  ```
  ports:
  - name: liveness-port
    containerPort: 8080
    hostPort: 8080

  livenessProbe:
      httpGet:
        path: /healthz
        port: liveness-port
      failureThreshold: 1
      periodSeconds: 10

  startupProbe:
    httpGet:
      path: /healthz
      port: liveness-port
    failureThreshold: 30
    periodSeconds: 10
  ```

** 참고: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

** 참고: 데브옵스를 위한 쿠버네티스 마스터
