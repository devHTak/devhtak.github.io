---
layout: post
title: Kubernetes ReplicaSet과 Deployment
summary: Kubernetes
author: devhtak
date: '2021-04-04 21:41:00 +0900'
category: Container
---

#### Replication Controller

- Kubernetes 1.8 이후 ReplicaSet으로 변경
- 지정한 replica 수의 POD Replica 가 실행중임을 보장한다. 즉, 레플리케이션 컨트롤러는 POD 또는 동일 종류의 POD의 Set이 항상 기동되고 사용 가능한지 확인한다
- Replication
  - 데이터 저장과 백업하는 방법과 관련이 있는 데이터를 호스트 컴퓨터에서 다른 컴퓨터로 복사하는 것
- 포드가 항상 실행되도록 유지하는 쿠버네티스 리소스
- 노드가 클러스터에서 사라지는 경우 해당 포드를 감지하고 대체 포드 생성
- 실행중인 포드의 목록을 지속적으로 모니터링 하고, '유형'의 실제 포드 수가 원하는 수와 항상 일치하는지 확인

- Replication Controller 의 세가지 요소
  - Replication Controller가 관리하는 포드 범위를 결정하는 레이블 셀렉터
  - 실행해야 하는 포드의 수를 결정하는 복제본 수
  - 새로운 포드의 모양을 설명하는 포드 템플릿

- Replication Controller 의 장점
  - 포드가 없는 경우 새 포드를 항상 실행
  - 노드에 장애 발생 시 다른 노드에 복제본 생성
  - 수동, 자동으로 수평 스케일링

- YAML 작성

  ```
  apiVersion: v1
  kind: ReplicationController
  metadata:
    name: http-go
  spec:
    replicas: 3
    selector:
      app: http-go
    template:
      metadata:
        name: http-go 
        labels:
          app: http-go
      spec:
        containers:
        - name: http-go 
          image: nginx
          ports:
          - containerPort: 80
  ```
  - replicas: 3
    - 실행해야 하는 포드의 수를 결정하는 복제본 수
  - selector.app: nginx
    - 레플리케이션 컨트롤러가 관리하는 포드 범위를 결정하는 레이블 셀렉터
  - template 하위
    - 새로운 포드의 모양을 설명하는 포드 템플릿
  - name과 app의 이름이 일치해야 한다.

- 실행 중인 레플리케이션컨트롤러와 포드 확인
  ```
  $ kubectl get pod
  NAME            READY   STATUS    RESTARTS   AGE
  http-go-4bfns   1/1     Running   0          61s
  http-go-j6zp8   1/1     Running   0          61s
  http-go-r68zp   1/1     Running   0          61s
  
  $ kubectl get rc
  NAME      DESIRED   CURRENT   READY   AGE
  http-go   3         3         3       75s
  ```
  
- 포드를 임의로 정지시켜 반응 확인
  ```
  $ kubectl delete pod http-go-4bfns
  pod "http-go-4bfns" deleted
  $ kubectl get pod
  NAME            READY   STATUS    RESTARTS   AGE
  http-go-88g46   1/1     Running   0          11s
  http-go-j6zp8   1/1     Running   0          2m23s
  http-go-r68zp   1/1     Running   0          2m23s
  ```
  - 일부로 delete한 후, 새로운 포드가 올라오는 지 확인
    - http-go-88g46 새로 생성
    
- 레플리케이션 정보 확인
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl describe rc http-go
  Name:         http-go
  Namespace:    default
  Selector:     app=http-go
  Labels:       app=http-go
  Annotations:  <none>
  Replicas:     3 current / 3 desired
  Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
  Pod Template:
    Labels:  app=http-go
    Containers:
     http-go:
      Image:        nginx
      Port:         80/TCP
      Host Port:    0/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Events:
    Type    Reason            Age    From                    Message
    ----    ------            ----   ----                    -------
    Normal  SuccessfulCreate  4m17s  replication-controller  Created pod: http-go-r68zp
    Normal  SuccessfulCreate  4m17s  replication-controller  Created pod: http-go-4bfns
    Normal  SuccessfulCreate  4m17s  replication-controller  Created pod: http-go-j6zp8
    Normal  SuccessfulCreate  2m5s   replication-controller  Created pod: http-go-88g46
  ```
  
- 노드 통신 직접 다운 시켜보기
  - 포드 사용 노드 확인
    ```
    server1@server1-VirtualBox:~/yaml$ kubectl get pod -o wide
    ```
  - work node 다운시키기
    ```
    $ gcloud compute ssh Worker2
    $ sudo ifconfig eth0 down
    (응답없음…)
    $ kubectl get node
    NAME    STATUS    ROLES   AGE   VERSION
    Master  Ready     <none>  10h   v1.12.8-gke.6
    Worker1 Ready     <none>  10h   v1.12.8-gke.6
    Worker2 NotReady  gpu     10h   v1.12.8-gke.6
    ```
  - Worker2에 있던 POD가 Running에서 업데이트 된다
    - 5분(Kubernetes default) 후 라는 시간을 준 것은 트래픽으로 인해 통신이 안되는 것을 감안한 것
    - Terminating이 되고 새로운 POD를 복구(생성) Pending 상태, ContainerCreating, Running 상태가 된다.
  
  - 네트워크 복구
    - 장애가 발생한 POD는 삭제한다.
    - 신규 POD는 장애가 생긴 Node가 아닌 현재 생성 가능한 Node에 생성한다.

- 레플리카컨트롤러 관리 레이블 벗어나기
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl label pod http-go-88g46 app-
  pod/http-go-88g46 labeled
  server1@server1-VirtualBox:~/yaml$ kubectl get pod --show-labels
  NAME            READY   STATUS    RESTARTS   AGE     LABELS
  http-go-64qrb   1/1     Running   0          21s     app=http-go
  http-go-88g46   1/1     Running   0          4m17s   <none>
  http-go-j6zp8   1/1     Running   0          6m29s   app=http-go
  http-go-r68zp   1/1     Running   0          6m29s   app=http-go
  nginx-test      1/1     Running   1          25h     app=nginx,team=dev2
  ```
  - 포드의 레이블이 변경되어 관리 밖으로 벗어나면 이를 건드리지 않고 새로운 포드를 생성한다.
    - http-go-88g46의 app 레이블 삭제했다.
    - http-go-64qrb을 새로 생성하여 관리한다.

- scaling 방법 1. 설정파일 수정하기
  ```
  $ kubectl edit rc http-go
  # Please edit the object below. Lines beginning with a '#' will be ignored,
  # and an empty file will abort the edit. If an error occurs while saving this file will be
  # reopened with the relevant failures.
  #
  apiVersion: v1
  kind: ReplicationController
  metadata:
    creationTimestamp: "2021-04-04T04:30:32Z"
    generation: 2
    labels:
      app: http-go
    name: http-go
    namespace: default
    resourceVersion: "26271"
    uid: e706c584-9897-4b96-9e00-db25ed11d502
  spec:
    replicas: 10
    selector:
      app: http-go
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: http-go
        name: http-go
      spec:
        containers:
        - image: nginx
          imagePullPolicy: Always
          name: http-go
          ports:
          - containerPort: 80
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 10

  ```
  - vi이 열리면spec.replicas 개수를 10에서 5개로 수정
  - 포드의 수 변화 확인
    ```
    server1@server1-VirtualBox:~/yaml$ kubectl get pod
    NAME            READY   STATUS    RESTARTS   AGE
    http-go-64qrb   1/1     Running   0          13m
    http-go-88g46   1/1     Running   0          17m
    http-go-f7n5t   1/1     Running   0          2m23s
    http-go-j6zp8   1/1     Running   0          19m
    http-go-r68zp   1/1     Running   0          19m
    ```
    
- scaling 방법 2. Replication Controller 설정 변경
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl scale rc http-go --replicas=10
  replicationcontroller/http-go scaled
  server1@server1-VirtualBox:~/yaml$ kubectl get pod
  NAME            READY   STATUS              RESTARTS   AGE
  http-go-64qrb   1/1     Running             0          11m
  http-go-6v2wx   0/1     ContainerCreating   0          4s
  http-go-88g46   1/1     Running             0          15m
  http-go-bbqnc   0/1     ContainerCreating   0          4s
  http-go-bt47n   0/1     ContainerCreating   0          4s
  http-go-f7n5t   0/1     ContainerCreating   0          4s
  http-go-j6zp8   1/1     Running             0          17m
  http-go-nhkvb   0/1     ContainerCreating   0          4s
  http-go-r9vjr   0/1     ContainerCreating   0          4s
  http-go-sgz6k   0/1     ContainerCreating   0          4s
  ```

- scaling 방법 3. 새로운 yaml 파일을 생성하여 적용하기
  ```
  $ cp replication-controller.yaml replication-controller-v2.yaml
  $ kubectl apply -f replication-controller-v2.yaml
  ```

- 삭제
  - 일반적인 삭제 명령어와 동일
  ```
  $ kubectl delete rc http-go
  ```
  
- 실행 시키고 있는 포드는 계속 실행을 유지하고 싶은 경우에는 --cascade 옵션 사용
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl delete rc http-go --cascade=false
  warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
  replicationcontroller "http-go" deleted
  server1@server1-VirtualBox:~/yaml$ kubectl get pod
  NAME            READY   STATUS    RESTARTS   AGE
  http-go-64qrb   1/1     Running   0          16m
  http-go-88g46   1/1     Running   0          20m
  http-go-f7n5t   1/1     Running   0          5m17s
  http-go-j6zp8   1/1     Running   0          22m
  http-go-r68zp   1/1     Running   0          22m
  ```
  
#### ReplicaSet

- 레플리카셋의 등장
  - 쿠버네티스 1.8부터 deployment, demonset, replicaset, statefulset 인 네개의 API가 베타로 업데이트되고, 1.9 버전에서는 정식 버전으로 업데이트 되었다.
  - ReplicaSet은 차세대 Replication Controller로 Replication Controller를 완전히 대체 가능
  - 초기 쿠버네티스에서 제공했기 때문에 여전히 Replication Controller 사용중인 경우도 존재

- Replication Controller VS ReplicaSet
  - ReplicaSet은 Replication Controller와 거의 동일하게 동작
  - ReplicaSet이 더 풍부한 표현식으로 포드 셀렉터 사용 가능하다.
    - Replication Controller: 특정 레이블을 포함하는 포드가 일치하는 지 확인
    - ReplicaSet: 특정 레이블이 없거나 해당 값과 관계업이 특정 레이블 키를 포함하는 포드를 매치하는지 확인

- ReplicaSet 생성
  - 대부분의 요소는 거의 비슷
  - apiVersion: apps/v1beta2
  - kind: ReplicaSet
  - matchExpressions: 레이블을 매칭하는 별도의 표현 방식 존대
    ```
    spec:
      replicas: 3
      selector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - http-go
    ```
    ```
    spec:
      replicas: 3
      selector:
        matchExpressions:
          app: http-go
    ```
    ```
    spec:
      replicas: 3
      selector:
        matchLabels:
          tier: some-tier
        matchExpressions:
        - {key: tier, operator: In, values: [some-tier]}        
    ```

- 레플리카셋의 조회와 삭제
  - 다른 리소스와 모두 동일
  - 조회
    ```
    $kubectl get rs
    ```
  - 상세 조회 
    ```
    $ kubectl describe rs http-go-rs
    ```
  - 삭제
    ```
    $ kubectl delete rs http-go-rs
    ```

- 예제
  - nginx 이미지로 rs-nginx 이름의 ReplicaSet 생성 후 10개로 scaling 
  - rs-nginx.yaml
    ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: rs-nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: rs-nginx
      template:
        metadata:
          labels:
            app: rs-nginx
        spec:
          containers:
          - name: nginx
            image: nginx
            ports:
              containerPort: 80
    ```
    ```
    $ kubectl create -f rs-nginx.yaml
    ```
  - 생성 확인
    ```
    $ kubectl get rs
    NAME       DESIRED   CURRENT   READY   AGE
    rs-nginx   3         3         3       38s
    $ kubectl get pod
    NAME             READY   STATUS    RESTARTS   AGE
    rs-nginx-lh59v   1/1     Running   0          22s
    rs-nginx-njwc6   1/1     Running   0          22s
    rs-nginx-vg7pj   1/1     Running   0          22s
    ```
  - 10개로 scaling
    - scale 명령
      ```
      $ kubectl scale rs rs-nginx --replicas=10
      replicaset.apps/rs-nginx scaled
      $ kubectl get pod
      NAME             READY   STATUS              RESTARTS   AGE
      rs-nginx-2s8bk   1/1     Running             0          23s
      rs-nginx-89nzk   1/1     Running             0          23s
      rs-nginx-jjmh9   0/1     ContainerCreating   0          23s
      rs-nginx-jtqjk   1/1     Running             0          23s
      rs-nginx-l9pvq   1/1     Running             0          23s
      rs-nginx-lh59v   1/1     Running             0          92s
      rs-nginx-mtvtd   0/1     ContainerCreating   0          23s
      rs-nginx-njwc6   1/1     Running             0          92s
      rs-nginx-s8z7m   1/1     Running             0          23s
      rs-nginx-vg7pj   1/1     Running             0          92s
      ```
    - yaml파일 작성 후 apply
      ```
      $ kubectl apply -f rs-nginx2.yaml 
      ```
    - edit 명령으로 rs 수정
      ```
      $ kubectl edit rs rs-nginx
      replicaset.apps/rs-nginx edited
      ```
    
#### Deployment

- 디플로이먼트(Deployment) 는 파드와 레플리카셋(ReplicaSet)에 대한 선언적 업데이트를 제공한다.
- 디플로이먼트에서 의도하는 상태 를 설명하고, 디플로이먼트 컨트롤러(Controller)는 현재 상태에서 의도하는 상태로 비율을 조정하며 변경한다. 
- 새 레플리카셋을 생성하는 디플로이먼트를 정의하거나 기존 디플로이먼트를 제거하고, 모든 리소스를 새 디플로이먼트에 적용할 수 있다.
- 애플리케이션을 다운 타임 없이 업데이트 가능하도록 도와주는 리소스
- 레플리카셋과 레플리케이션 컨트롤러 상위의 배포되는 리소스
- Deployment -> ReplicaSet or Replication Controller -> POD

- 모든 포드를 업데이트 하는 방법
  - 잠깐의 다운 타임 발생, 새로운 포드를 실행시키고 작업이 완료되면 오래된 포드를 삭제
  - 롤링 업데이트

- Deployment 작성 요령
  - 포드의 metadata 부분과 spec 부분을 그대로 옮김
  - Deployment의 spec.template에는 배포할 포드를 설정
  - replicas에는 이 포드를 몇개를 배포할 지 명시
  - label은 deployment가 배포한 포드를 관리하는 데 사용
    
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
  ```
  
 - Deployment scaling
  - yaml 파일 직접 수정하여 replicas 수 조정
    ```
    $ kubectl edit deploy <deploy name>
    ```
  - --replicas 옵션으로 replicas 수 조정
    ```
    $ kubectl scale deploy <deploy name> --replicas=<Number>
    ```
    
  
** 참조: 데브옵스(DevOps)를 위한 쿠버네티스 마스터
