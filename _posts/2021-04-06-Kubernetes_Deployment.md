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


** 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터
