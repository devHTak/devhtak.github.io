---
layout: post
title: Kubernetes Deployment
summary: Kubernetes
author: devhtak
date: '2021-04-06 21:41:00 +0900'
category: Container
---

#### Deployment

- 디플로이먼트(Deployment) 는 파드와 레플리카셋(ReplicaSet)에 대한 선언적 업데이트를 제공한다.
  - 빠르게 apiVersion을 업데이트할 수 있다.
  - 애플리케이션의 업데이트와 배포를 더욱 편하게 만들어준다.
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

- 연습문제, jenkins 디플로이먼트를 deploy-jenkins를 생성하라
 - app: jenkins-test로 레이블링
   - deploy-jenkins.yaml
     ```
     apiVersion: apps/v1
     kind: Deployment
     metadata
       name: deploy-jenkins
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: jenkins-test
       template:
         metadata:
           labels:
             app: jenkins-test
         spec:
           containers:
           - name: jenkins
             image: jenkins/jenkins
             ports:
             - containerPort: 8080
     ```
    - 확인
      ```
      server1@server1-VirtualBox:~/yaml$ kubectl get deploy
      NAME             READY   UP-TO-DATE   AVAILABLE   AGE
      depoly-jenkins   3/3     3            3           3m23s
      server1@server1-VirtualBox:~/yaml$ kubectl get rs
      kNAME                        DESIRED   CURRENT   READY   AGE
      depoly-jenkins-7ccbbccdc5   3         3         3       3m27s
      server1@server1-VirtualBox:~/yaml$ kubectl get pod
      NAME                              READY   STATUS    RESTARTS   AGE
      depoly-jenkins-7ccbbccdc5-5ggxq   1/1     Running   0          3m30s
      depoly-jenkins-7ccbbccdc5-cnkcz   1/1     Running   0          3m30s
      depoly-jenkins-7ccbbccdc5-r8qls   1/1     Running   0          3m30s
      ```

#### Rolling Update와 Rollback

- 기존의 모든 포드를 삭제 후 새로운 포드 생성
  - 기존 서비스 업데이트를 진행할 때에는 잠깐의 서비스 다운 타임이 발생했다.

- Rolling Update 사용
  - 새로운 포드를 실행시키고 작업이 완료되면 오래된 포드를 삭제
    - 새 버전을 실행하는 동안 구 버전 포드와 연결
    - 서비스의 레이블 셀렉터를 수정하여 간단하게 수정 가능

- ReplicaSet이 제공하는 Rolling Update
  - 이전에는 kubectl을 사용해 스케일링을 사용하여 수동으로 롤링 업데이트 진행 가능
    - kubectl -> Replication Controller 다수 -> POD 다수
    - 수동으로 작업하면 Human Error, 노가다 등의 문제 발생
  - kubectl이 중단되면 업데이트는 어떻게 될까??    
  - ReplcaSet 또는 Replicatoin Controller을 통제할 수 있는 시스템이 필요하다.
    
- Deployment 로 해결
  - Label Selector, 원하는 복제본 수(replicas), 포드 템플릿
  - 디플로이먼트의 전략은 yaml에 지정하여 사용 가능
  - 먼저 업데이트 시나리오를 위해 3개의 도커 이미지 준비
    - gasbugs/http-go:v1
    - gasbugs/http-go:v1
    - gasbugs/http-go:v1
    
  - YAML 만들기
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: http-go-deployment
      labels:
        app: http-go
    spec:
      replicas: 3
      template:
        metadata:
          name: http-go
          labels:
            app: http-go
        spec:
          containers:
          - image: gasbugs/http-go:v1
            name: http-go
    ```
  - deploy 생성
    ```
    $ kubectl create -f http-go-deployment.yaml --record=true
    ```
    - history에 백업하기 위해서 --record=true 옵션을 주었다.
  
  - deployment 확인
    ```
    $ kubectl get deployment
    NAME      READY   UP-TO-DATE   AVAILABLE   AGE
    http-go   0/3     3            0           76s
    ```
    
  - ReplicaSet 확인
    ```
    $ kubectl get rs
    NAME                DESIRED   CURRENT   READY   AGE
    http-go-ccb794f48   3         3         0       97s
    ```
    
  - POD 확인
    ```
    $ kubectl get pod
    NAME                      READY   STATUS              RESTARTS   AGE
    http-go-ccb794f48-5md8p   0/1     ContainerCreating   0          111s
    http-go-ccb794f48-cwqtj   0/1     ContainerCreating   0          111s
    http-go-ccb794f48-wht5s   0/1     ContainerCreating   0          111s
    ```
    
  - deploy 상세 설정 확인
    ```
    $ kubectl describe deploy http-go
    Name:                   http-go
    Namespace:              default
    CreationTimestamp:      Thu, 08 Apr 2021 14:46:21 +0900
    Labels:                 app=http-go
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               app=http-go
    Replicas:               3 desired | 3 updated | 3 total | 0 available | 3 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=http-go
      Containers:
       http-go:
        Image:        gasbugs/http-go:v1
        Port:         8080/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      False   MinimumReplicasUnavailable
      Progressing    True    ReplicaSetUpdated
    OldReplicaSets:  <none>
    NewReplicaSet:   http-go-ccb794f48 (3/3 replicas created)
    Events:
      Type    Reason             Age    From                   Message
      ----    ------             ----   ----                   -------
      Normal  ScalingReplicaSet  3m38s  deployment-controller  Scaled up replica set http-go-ccb794f48 to 3
    ```
  
  - rollout을 통해 상태 확인
    ```
    $ kubectl rollout status deploy http-go
    deployment "http-go" successfully rolled out    
    ```
  - history 확인
    ```
    $ kubectl rollout history deploy http-go
    deployment.apps/http-go 
    REVISION  CHANGE-CAUSE
    1         kubectl create --filename=http-go-deploy.yaml --record=true
    ```
  
    
- Deployment update 전략
  - Rolling Update (기본값)
    - 오래된 포드를 하나씩 제거하는 동시에 새로운 포드 추가
    - 요청을 처리할 수 있는 양은 그대로 유지
    - 반드시 이전 버전과 새 버전을 동시에 처리 가능하도록 설계한 경우에만 사용
    ```
    spec:
      strategy:
        type: RollingUpdate
    ```
    
  - Recreate
    - 새 포드를 만들기 전에 이전 포드를 모두 삭제
    - 여러 버전을 동시에 실행 불가능
    - 잠깐의 다운 타임 존재

  - 업데이트 과정을 보기 위한 속도 조절
    ```
    $ kubectl patch deployment http-go -p {"spec": {"minReadySeconds": 10}}
    ```
    - minReadySeconds: 업데이트 준비를 10초 동안 갖게 된다.
  
  - Load Balancer를 위한 expose
    ```
    $ kubectl expose deploy http-go
    service/http-go exposed
    $ kubectl get svc
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    http-go      ClusterIP   10.107.200.227   <none>        8080/TCP   6s
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    4m52s
    ```
    
- 디플로이먼트 업데이트 실행
  - 새로운 터미널을 열어 이미지 업데이트 실행
    ```
    $ kubectl set image deployment http-go http-go=gasbugs/http-go:v2
    deployment.apps/http-go image updated
    kubectl rollout history deploy http-go
    deployment.apps/http-go 
    REVISION  CHANGE-CAUSE
    1         kubectl create --filename=http-go-deploy.yaml --record=true
    2         kubectl create --filename=http-go-deploy.yaml --record=true
    ```
    - http-go의 이미지를 v2로 set
    - 두번째 revision이 추가되었다.
  
  - 모니터링하는 시스템 관찰
    ```
    $ while true; curl 10.107.200.227; sleep 1; done
    ```
    
- 디플로이먼트 업데이트 실행 결과
  - 업데이트한 이력 확인
    - Revision의 개수는 디폴트로 10개까지 저장
      ```
      $ kubectl rollout history deployment http-go
      ```
      
- rollback 실행하기
  - 롤백을 실행하면 이전 업데이트 상태로 돌아감
  - 롤백을 하여도 히스토리의 리비전 상태는 이전 상태로 돌아가지 않음
  - 이미지 수정
    ```
    $ kubectl edit deploy http-go # iamge 버전 수정
    deployment.apps/http-go edited
    $ kubectl rollout history deploy http-go
    deployment.apps/http-go 
    REVISION  CHANGE-CAUSE
    1         kubectl create --filename=http-go-deploy.yaml --record=true
    2         kubectl create --filename=http-go-deploy.yaml --record=true
    3         kubectl create --filename=http-go-deploy.yaml --record=true
    ```
  - 롤백하기
    ```
    $ kubectl rollout undo deploy http-go
    deployment.apps/http-go rolled back
    $ kubectl rollout history deploy http-go
    deployment.apps/http-go 
    REVISION  CHANGE-CAUSE
    1         kubectl create --filename=http-go-deploy.yaml --record=true
    3         kubectl create --filename=http-go-deploy.yaml --record=true
    4         kubectl create --filename=http-go-deploy.yaml --record=true
    ```
    - 특정 버전으로 rollback하기
      ```
      $ kubectl rollout undo deploy http-go --to-revision=1
      ```
      - revision 1로 롤백
        
- 롤링 업데이터 전략 세부 설정
  ```
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  ```
  - maxSurge
    - 기본값 25%, 개수로도 설정 가능
    - 최대로 추가 배포를 허용할 개수 설정
    - 4개인 경우 25%이면, 1개가 설정(총 개수 5개까지 동시 포드 운영)

  - maxUnavailable
    - 기본값 25%, 개수로도 설정 가능
    - 동작하지 않는 포드의 개수 설정
    - 4개인 경우 25%이면, 1개가 설정(총 개수 4-1개는 운영해야 함)
  
  ```
  spec:
    strategy:
      rollingUpdate:
        maxSurge: 1
        maxUnavaliable: 1
      type: RollingUpdate
  ```
  
- 롤아웃 일시중지와 재시작
  - 업데이트 중에 일시정지하길 원하는 경우
    ```
    $ kubectl rollout pause deployment http-go
    ```
    
  - 업데이트 일시중지 중 취소
    ```
    $ kubectl rollout undo deployment http-go
    ```
    
  - 업데이트 재시작
    ```
    $ kubectl rollout resume deployment http-go
    ```
    
- 업데이트를 실패한 경우
  - 업데이트를 실패하는 케이스
    - 부족한 할당량 (Insufficient quota)
    - 레디네스 프로브 실패(Readiness probe failures)
    - 이미지 가져오기 오류(Image pull errors)
    - 권한 부족(Insufficient permissions)
    - 제한 범위(Limit ranges)
    - 응용 프로그램 런타임 구성 오류(Application runtime misconfiguration)

  - 업데이트를 실패하는 경우 기본적으로 600초 후에 업데이트 중지
    ```
    spec:
      processDeadlineSeconds: 600
    ```

** 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터
