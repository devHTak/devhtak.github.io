---
layout: post
title: Kubernetes label과 selector
summary: Kubernetes
author: devhtak
date: '2021-04-02 21:41:00 +0900'
category: Container
---

#### label이란?

- 모든 리소스를 구성하는 매우 간단하면서도 강력한 쿠버네티스 기능
- 리소스에 첨부하는 임의의 키/값 쌍 (ex. app:test)
- 여러 객체에 태그를 붙여 구분할 수 있도록 함
- 레이블 셀렉터를 사용하면 각종 리소스를 필터링하여 선택할 수 있다.
- 리소스는 한 개 이상의 레이블을 가질 수 있음
- 리소스를 만드는 시점에 레이블을 첨부
- 기존 리소스에도 레이블의 값을 수정 및 추가 기능
- 모든 사람이 쉽게 이해할 수 있는 체계적인 시스템을 구축 가능
  - app: 애플리케이션 구성요소, 마이크로서비스 유형 지정
  - rel: 애플리케이션의 버전 지정

#### label을 이용한 POD 구성

- 쿠버네티스 인 액션에서 사용한 예제
![image](https://user-images.githubusercontent.com/42403023/113420419-43034580-9404-11eb-8576-d63784b61a92.png)
  - 가로축은 app을 key로 구성하여 microservice로 나누어준 POD
  - 세로축은 rel을 key로 구성하여 환경을 나눠주고 있다.
- 참조: https://livebook.manning.com/book/kubernetes-in-action/chapter-3/141

#### POD 생성 시 레이블을 지정하는 방법

- go-test-v2.yaml 작성
  - labels: creation_method / env 지정  
    ```
    server1@server1-VirtualBox:~/yaml$ vi go-test-v3.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: go-test-v2
      labels:
        creation_method: manual
        env: prod
    spec:
      containers:
      - image: devtak/test:v1.0
        name: go-test-v2
        ports:
        - containerPort: 8080
          protocol: TCP
    ```
- POD 생성
  ```
  server1@server1-VirtualBox:~/yaml$ kubectl create -f go-test-v2.yaml
  ```

- 레이블을 추가 및 수정하는 방법
  - 새로운 레이블을 추가할 때는 label 명령어를 사용
    ```
    $ kubectl label pod go-test test=foo
    ```
    - 기존에 있던 go-test POD에 test=foo label 추가
  
  - 기존의 레이블을 수정할 때는 --overwrite 옵션을 주어서 실행
    ```
    $ kubectl label pod go-test-v2 rel=beta
    error: 'rel' already has a value (canary), and --overwrite is false
    $ kubectl label pod go-test-v2 rel=beta --overwrite
    pod/go-test-v2 labeled
    ```
    - 기존에 go-test-v2 POD에 있는 rel 키값에 value를 beta로 overwrite한다.
    - 해당 key가 있는 경우 --overwrite 옵션을 주어야 한다.
    
  - 레이블 삭제
    ```
    $ kubectl label pod go-test-v2 rel-
    ```
    - key값에 -(minus)를 표시하면 삭제 가능하다.

- 레이블 확인하기
  - 레이블 보여주기
    ```
    $ kubectl get pod --show-labels
    NAME READY STATUS RESTARTS AGE LABELS
    go-test 1/1 Running 0 53m <none>
    go-test-v2 1/1 Running 0 2m5s app=go-test,foo=bar,rel=beta,test=foo
    ```
    
  - 특정 레이블 컬럼으로 확인
    ```
    $ kubectl get pod -L app,rel
    NAME      READY STATUS    RESTARTS AGE  APP     REL
    go-test    1/1  Running   0        62m
    go-test-v2 1/1  Running   0        11m  go-test beta
    ```
    - 없어도 빈 값으로 보여진다.
    
- 레이블로 필터링하여 검색
  - -l 옵션을 확인해서 필터링 기능을 사용할 수 있다.
  - env label을 갖는 POD 확인
    ```
    $ kubectl get pod --show-labels -l 'env' 
    NAME        READY   STATUS    RESTARTS AGE    LABELS
    go-test-v2  1/1     Running   0        3m10s  creation_method=manual,env=prod,rel=beta,test=foo
    ```
  - env label을 갖지 않는 POD 확인
    ```
    $ kubectl get pod --show-labels -l '!env'
    NAME      READY   STATUS    RESTARTS    AGE   LABELS
    go-test   1/1     Running   0           2m    <none>
    ```
  - env label이 test가 아닌 POD 확인
    ```
    $ kubectl get pod --show-labels -l 'env!=test'
    NAME        READY   STATUS    RESTARTS  AGE     LABELS
    go-test     1/1     Running   0         4m      <none>
    go-test-v2  1/1     Running   0         5m14s   creation_method=manual,env=prod,rel=beta,test=foo
    ```
  - env label이 test가 아니고 rel label이 beta인 POD 확인
    ```
    $ kubectl get pod --show-labels -l 'env!=test,rel=beta'
    NAME        READY   STATUS    RESTARTS  AGE     LABELS
    go-test-v2  1/1     Running   0         5m58s   creation_method=manual,env=prod,rel=beta,test=foo
    ```

- 레이블 배치 전략
  - 확장 가능한 쿠버네티스 레이블 예제
    
    |label key|description|label value|
    |---|---|---|
    |Application-ID/Application-name|응용 프로그램 이름 또는 ID|my-awesome-app/app-nr-2345|
    |Version-nr|버전 번호|ver-0.9|
    |Owner|개체가 속한 팀 또는 개인|Team-kube/Josh|
    |Stage/Phase|개발 단계 또는 위치|Dev, staging, QA, Canary, Production|
    |Release-nr|릴리즈 번호|release-nr-2.0.1|
    |Tier|앱이 속한 계층|front-end/back-end|
    |Customer-facing|고객에게 직접 서비스 하는 앱 여부|Yes/No|
    |App-role|앱의 역할|Cache/Web/Database/Auth|
    |Project-ID|연관된 프로젝트 ID|my-project-276|
    |Customer-ID|자원을 할당한 고객 ID|customer-id-29|
    
** 출처: 데브옵스를 위한 쿠버네티스 마스터

** 출처: 쿠버네티스 인 액션
