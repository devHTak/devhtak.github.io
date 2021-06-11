---
layout: post
title: Kubernetes Security
summary: Kubernetes
author: devhtak
date: '2021-06-11 21:41:00 +0900'
category: Container
---

#### 보안을 위한 다양한 리소스

- 모든 통신은 TLS
  - 대부분 엑세스는 kube-apiserver를 통하지 않고서는 불가능하다.
  - 엑세스 가능한 유저
    - 파일 - 유저 이름과 토큰
    - Service-Accounts
    - 인증서(Certificates)
    - External Authentication Providers - LDAP
    
  - 무엇을 할 수 있는가?
    - RBAC Authorization
    - ABAC Authorization
    - Node Authorization
    - WebHook Mode

#### 스태틱토큰과 서비스어카운트

- Acccounts
  - Accounts는 두가지의 타입이 존재
    - 사용자를 위한 user
    - 애플리케이션(포드 외)을 위한 service account
  
  - 스태틱 토큰 파일
    - Apiserver 서비스를 실행할 때 --token-auth-file=SOMEFILE.csv 전달 (kube-apiserver 수정 필요)
      - SOMEFILE.csv 예시
        ```
        $ sudo vi SOMEFILE.csv 
        password1,user1,uid1,"group1"
        password2,user2,uid2
        ```
        - csv 는 띄어쓰기 없이 해야 한다.
        ```
        $ sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
        # ...
        spec:
          containers:
          - command:
            - --token-auth-file=/etc/kubernetes/pik/SOMEFILE.csv
        # ...
        ```
    - API Server를 다시 시작해야 적용됨
      ```
      $ docker ps -a | grep api
      $ docker logs container-id
      $ sudo cp somefile.csv /etc/kubernetes/pik
      ```
      - 바로 적용되지 않고 오류가 발생한다
      - hostPath를 추가하거나 기존 등록되어 있는 mountPath 경로에 SOMEFILE.csv 파일 입력
      
    - 토큰, 사용자 이름, 사용자 UID, 선택적으로 그룹 이름 다음에 최소 3열의 csv 파일
    
  - Static token file을 적용했을 때 사용방법
    - HTTP 요청을 진행할 때 다음과 같은 내용을 헤더에 포함해야 함
    - Authorization: Bearer 31ada...
      ```
      $ TOKEN=password1
      $ APISERVER=https://127.0.0.1:6443
      $ curl -X GET $APISERVER/api --header "Authorization Bearer $TOKEN" --insecure
      ```
    - kubectl에 등록하고 사용하는 방법
      ```
      $ kubectl config set-credentials user1 --token=password1
      $ kubectl config set-context user1-context --cluster=kubernetes \
          --namespace=frontend --user=user1
      $ kubectl get pod --user user1      
      ```
  - Service Account 만들기
    - 포드에 별도의 설정을 주지 않으면 기본적으로 service account가 생성
      ```
      $ kubectl get sa default -o yaml
      ```
    - 명령어를 사용하여 serviceaccount sa1을 생성
      ```
      kubectl create serviceaccount sa1
      ```
    - Pod에 spec.serviceAccount: service-account-name 과 같은 형식으로 지정

#### TLS 인증서 활용

- SSL 통신 과정 이해
  - 응용계층인 HTTP와 TCP 계층 사이에서 작동
  - 애플리케이션에 독립적 -> HTTP 제어를 통한 유연성
  - 데이터의 암호화(기밀성), 데이터 무결성, 서버인증기능, 클라이언트 인증 기능
  
  ![image](https://user-images.githubusercontent.com/42403023/121617467-f1300900-ca9f-11eb-8f6a-47a009967d56.png)
  
  - 이미지 출처: DevOps 를 위한 쿠버네티스 마스터 강의 중
  
- Certificate 를 보장하는 방법: 인증기관(CA)
  
  ![image](https://user-images.githubusercontent.com/42403023/121617567-39e7c200-caa0-11eb-92b0-d4f5848b6714.png)
  
  - 이미지 출처: https://docs.pexip.com/admin/certificate_managent.html

- kubernetes의 인증서 위치
  ```
  $ sudo ls /etc/kubernetes/pki
  
  $ sudo ls /etc/kubernetes/pki/etcd
  ```
  
- 정확한 TLS 인증서 사용 점검
  - 적절한 키를 사용하는지 확인하려면 manifests 파일에서 실행하는 certificate 확인 필요
  - 적절한 키를 사용하지 않으면 에러 발생
    ```
    $ sudo ls /etc/kubernetes/manifests
    ```

#### 출처

- DevOps 를 위한 쿠버네티스 마스터 강의
