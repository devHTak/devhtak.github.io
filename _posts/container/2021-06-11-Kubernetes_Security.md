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

#### TLS 인증서 정보 확인과 자동갱신 방법

- 인증서 정보 확인
  ```
  $ sudo openssl x509 -in <certificate> -text
  ```

- 인증서 갱신
  - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs
  - Check certificate expiration
    ```
    $ kubeadm alpha certs check-expiration
    ```
  - Automatic certificate renewal
    ```
    kubeadm 은 컨트롤 플레인을 업그레이드하면 모든 인증서를 자동 갱신
    ```
  - Manual certificate renewal
    ```
    $ kubeadm alpha certs renew all
    ```

#### TLS 인증서를 활용한 유저 생성

- ca를 사용하여 직접 csr(certificate sending request) 승인하기
  - 개인 키 생성
    ```
    $ openssl genrsa -out ${random.key} 2048
    ```
  - private 키를 기반으로 인증서 서명 요청하기
    - CN: 사용자 이름
    - O: 그룹 이름
    - CA에게 CSR 파일로 인증을 요청할 수 있음
    ```
    $ openssl req -new -key ${random.key} -out ${csr} -subj
    "/CN={}/O={}"
    ```
    
  - ca를 사용하여 직접 csr 승인하기
    - kubernetes 클러스터 인증 기관(CA) 사용이 요청을 승인해야 함
    - 내부에서 직접 승이하는 경우 pki 디렉토리에 있는 ca.key, ca.crt를 사용하여 승인 가능
    - csr을 승인하여 최종 인증서를 생성
    - -days 옵션을 사용해 며칠간 인증서가 유효할 수 있는지 설정
    ```
    $ openssl x509 -req -in ${csr} -CA /etc/kubernetes/pki/ca.cert =CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ${crt} -days 500
    ```

  - 인증 사용을 위해 쿠버네티스에 crt를 증록
    - crt를 사용할 수 있도록 kubectl을 사용하여 등록
      ```
      $ kubectl config set-credentials ${name} --client-certificate=.${crt} -- client-key=${key}
      ```
    - 다음 명령을 사용하여 사용자 권한으로 실행 가능 
      - 지금은 사용자에게 권한을 할당하지 않아 실행되지 않음
      ```
      $ kubectl --context=gasbugs-context get pods
      ```

#### kubeconfig 설정 관리
  
- 직접 curl을 사용하여 요청
  - key, cert, cacert 키를 가지고 직접 요청 가능
  - 그러나 매번 이 요청을 사용하기에는 무리가 있음
    ```
    $ curl https://kube-api-server:6443/api/v1/pods \
    --key user.key \
    --cert user.crt \
    --cacert ca.crt \
    ```

- kube config 파일 확인하기
  ```
  $ kubectl config view
  $ kubectl config view --kube config=<config file>
  ```

- kube config 의 구성
  - ~/.kube/config 파일을 확인하면 세가지 부분으로 작성됨
  - clusters: 연결할 쿠버네티스 클러스터의 정보 입력
  - users: 사용할 권한을 가진 사용자 입력
  - contexts: cluster와 user를 함께 입력하여 권한 할당
  - 예시
    - curl에 입력되는 parameter를 config 파일에 세부분으로 나누어 작성이 가능하다.
    ```
    $ curl https://kube-api-server:6443/api/v1/pods \
    --key user.key \
    --cert user.crt \
    --cacert ca.crt \
    ```
    ```
    # kubectl config view
    # Clusters
    name: kube-cluster
    cluster:
      certificate-authority: ca.crt
      server: https://cluster-ip:6443
    # Contexts
    name: user-name@kube-cluster
    context:
      cluster: kube-cluster
      user: user-name
    # Users
    name: user-name
    user:
      client-certificate: user.crt
      client-key: user.key
    ```
  - context는 cluster와 user로 구성되어 있다.
  - 인증 사용자 바꾸기
    ```
    $ kubectl config use-context user@kube-cluster
    ```
  - 인증 테스트
    ```
    $ kubectl get pod
    # forbidden이 나오면 성공
    ```
    
#### RBAC를 활용한 롤 기반 액세스 컨트롤

- 역할 기반 액세스 제어(RBAC)
  - 기업 내에서 개별 사용자의 역할을 기반으로 컴퓨터, 네트워크 리소스에 대한 액세스를 제어
  - rbac.authorization.k8s.ioAPI를 사용하여 정의
  - 권한 결정을 내리고 관리자가 Kubernetes API를 통해 정책을 동적으로 구성
  - RBAC를 사용하여 롤을 정의하려면 apiserver에 --authorization-mode=RBAC 옵션 필요
    - kubeadm을 기본으로 설치되어 있으면 해당 옵션이 저장되어 있다.

- rbac.authorization.k8s.ioAPI
  - RBAC를 다루는 이 API는 총 4가지의 리소스를 컨트롤
    - Role
    - RoleBinding
    - ClusterRole
    - ClusterRoleBinding

  - Role과 Rolebinding의 차이
    - Role
      - "누가"하는 것인지는 정하지 않고 Rule(룰)만을 정의
      - 일반 롤의 경우에는 네임스페이스 단위로 역할 관리
      - 클러스터롤은 네임스페이스 구애 받지 않고 특정 자원을 전체 클러스터에서 자원을 관리할 롤을 정의
    - RoleBinding
      - "누가"하는 것인지만 정하고 롤은 정의하지 않음
      - 롤을 정의하는 대신에 참조할 롤을 정의(roleRef)
      - 어떤 사용자에게 어떤 권한을 부여할 지 정하는(바인딩) 리소스
      - 클러스터롤에는 클러스터롤바인딩, 일반 롤에는 롤바인딩 필요
    - 리소스 <-> Role(cluster) <-> RoleBinding(Cluster) <-> 사용자
  
  - Role 작성 요령
    ```
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      namespace: office
      name: deployment-manager
    rules:
    - apiGroups: ["", "extensions", "apps"] # 비어 있으면 cores라는 그룹에 속한다.
      resources: ["deployments", "replicasets", "pods"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    ```
    - role 생성
      ```
      $ kubectl create -f pod-reader-role.yaml
      ```
    - apiGroups는 resources마다 다르다
      - 테스트 해보면 가능하다.
        ```
        $ kubectl get pod --user gasbugs -n office
        # 권한에 대한 오류 메시지에 apiGroup, resources에 대한 내용이 포함되어 있다.
        ```
    
  - Rule에는 API 그룹, 리소스, 작업 가능한 동작 작성
    - API 를 사용하게 할 그룹 정의

- 사용자 권한 할당
  - 롤바인딩을 사용하여 office namespace 권한 할당
  - kubectl create -f 를 사용하여 생성
  - 해당 네임스페이스에 모든 권한을 생성
    ```
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/vibeta1
    metadata:
      name: deployment-manager-binding
      namespace: office
    subjects:
    - kind: User
      name: gasbugs
      apiGroup: ""
    roleRef:
      kind: Role
      name: deployment-manager
      apiGroup: ""
    ```
    - 생성
      ```
      $ kubectl create -f pod-reader-role-binding.yaml
      ```
    - office namespace 내에 User중 gasbugs를 선택
    - User가 아닌 ServiceAccount로 변경하여 사용할 수 있다.
    - roleRef에는 생성한 Role을 매핑해준다.
    
- 사용자 권한 테스트
  - 실행돼야 하는 명령어
    ```
    $ kubectl --context=gasbugs-context get pods -n office
    $ kubectl --context=gasbugs-context run --generator=pod-run/v1 nginx --image=nginx -n office
    ```
  - 실행되면 안되는 명령어
    ```
    $ kubectl --context=gasbugs-context get pods
    $ kubectl --context=gasbugs-context run --generator=pod-run/v1 nginx --image=nginx
    ```
    - namespace가 default이기 때문에 실행되면 안된다.

- ClusterRole과 ClusterRoleBinding
  - clusterrole 확인하기
    ```
    $ kubectl get clusterrole cluster-admin -o yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
      creationTimestamp: "2021-04-01T08:02:23Z"
      labels:
        kubernetes.io/bootstrapping: rbac-defaults
      name: cluster-admin
      resourceVersion: "84"
      uid: 9a8c8420-5d2c-49ba-bf11-c755cba5d263
    rules:
    - apiGroups:
      - '*'
      resources:
      - '*'
      verbs:
      - '*'
    - nonResourceURLs:
      - '*'
      verbs:
      - '*'
    ```
    - cluster-admin에 대한 yaml 확인 가능
    - 모든 권한을 가지고 있다.

  - clusterrolebinding 확인하기
    ```
    $ kubectl get clusterrolebinding cluster-admin -o yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
      creationTimestamp: "2021-04-01T08:02:24Z"
      labels:
        kubernetes.io/bootstrapping: rbac-defaults
      name: cluster-admin
      resourceVersion: "145"
      uid: d8ed66d9-a365-442c-ac37-bf0b6ef5a776
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: Group
      name: system:masters
    ```
  
- 권한 종류
  - https://kubernetes.io/docs/reference/access-authn-authz/authorization
  
  |HTTP Verb|request verb|
  |---|---|
  |POST|create|
  |GET, HEAD|get(for individual resources), list(for collections, including full object content), watch(for watching an individual resource or collection of resources)|
  |PUT|update|
  |PATCH|patch|
  |DELETE|delete(for individual resources), deletecollection(for collections)|
  
#### Security Context

- 보안 컨텍스트
  - 서버 침해사고 발생 시 침해사고를 당한 권한을 최대한 축소하여 그 사고에 대한 확대 방지
  - 침해사고를 당한 서비스가 모든 권한(루트나 노드 커널 기능)으로 동작하는 경우 서비스를 탈취한 공격자는 그대로 컨테이너의 권한을 사용할 수 있음
  - 최소 권한 정책에 따른 취약점 감소
  - 보안 컨텍스트 설정에는 다음 사항 포함
    - 권한 상승 기능 여부
    - 프로세스 기본 UID/GID 를 활용한 파일등의 오브젝트의 액세스 제어
    - Linux Capabilities 를 활용한 커널 기능 추가
    - 오브젝트에 보안 레이블에 지정하는 SELinux(Security Enhanced Linux) 기능
    - AppArmor: 프로그램 프로필을 사용하여 개별 프로그램 기능 제한
    - Seccomp: 프로세스의 시스템 호출 필터링

- 포드에 시큐리티 컨텍스트 설정
  - 포드에 UID를 설정하면 모든 컨테이너에 적용된다.
    - 컨테이너 UID/GID 설정
    - 권한상승 제한
  
  - security-context.yml
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: security-context-demo
    spec:
      securityContext: # id 지정
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
      volumes:
      - name: sec-ctx-vol
        emptyDir: {}
      containers:
      - name: sec-ctx-demo
        image: busybox
        command: ["sh", "-c", "sleep 1h"]
        volumeMounts:
        - name: sec-ctx-vol
          mountPath: /data/demo
        securityContext:
          allowPrivilegeEscalation: false
    ```
  - 실행
    ```
    $ kbuectl create -f security-context.yaml
    $ kubectl exec -it security-context-demo /bin/bash 
    / $ id
    uid=1000 gid=3000 groups=2000
    / $ ps -eaf
    PID USER  TIME  COMMAND
    1   1000  0:00  sleep 1h
    10  1000  0:00  sh
    16  1000  0:00  ps -eaf
    ```
    
- 컨테이너에 시큐리티 컨텍스트 설정
  - 컨테이너 내부에 새로운 유저를 사용하면 그 설정 우선
  - 컨테이너에 UID/GID 설정
  - security-context2.yaml
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: security-context-demo-2
    spec:
      securityContext:
        runAsUser: 1000
      containers:
      - name: sec-ctx-demo-2
        image: gcr.io/google-samples/node-hello:1.0
        securityContext:
          runAsUser: 2000
          allowPrivilegeEscalation: false
    ```

- 캐퍼빌리티 적용
  - 캐퍼빌리티를 적용하면 리눅스 커널에서 사용할 수 있는 권한을 추가 설정 가능
  - 캐퍼빌리티를 활용한 리눅스 커널 권한 변수
    - https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h
  - date+%T -s "12:00:00" 명령은 SYS_TIME 커널 권한이 있어야만 가능
  - security-context-4.yaml
    ```
    apiVersion: v1
    kind: Pod
    metadata: security-context-demo-4
    spec:
      containers:
      - name: sec-ctx-4
        image: gcr.io/google-samples/node-hello:1.0
        securityContext:
          capabilities:
            add: ["NET_ADMIN", "SYS_TIME"] # NET_ADMIN 과 SYS_TIME에 대하여 추가
    ```

- SELinux 권한
  - 모든 프로세스와 객체에 시큐리티 콘텍스트 (또는 레이블)을 달아 관리하는 형태
  - 네필터와 더불어 리눅스의 핵심 보안 기능 담당
  - 그러나 실무에서는 복잡도가 높아 오히려 소외
  - SELinux에 대한 자세한 정보 사이트
    - https://www.lesstif.com/ws/selinux/selinux
  - 다음과 같은 형태로 취급
    ```
    # ...
    securityContext:
      seLinuxOptions:
        level: "s0:c123,c456"
    ```
  
  |요소|설명|
  |---|---|
  |사용자|시스템의 사용자와는 별도의 SELinux 사용자로 역할이나 레벨과 연계하여 접근 권한을 관리하는 데 사용|
  |역할(Role)|하나 혹은 그 이상의 타입과 연결되어 SELinux의 사용자의 접근을 허용할 지 결정하는 데 사용|
  |타입(Type)|Type Enforcement의 속성중 하나로 프로세스의 도메인이나 파일의 타입을 지정하고 이를 기반으로 접근 통제 수행|
  |레이블(Label)|레이블은 MLS(Multi Level System)에 필요하여 강제 접근 통제보다 더 강력한 보안이 필요할 때 사용하는 기능으로 정부나 군대 등 최고의 기빌을 취급하는 곳이 아니면 사용하지 않음|

#### 네트워크 정책(Network Policy)

- 네트워크 정책
  - 포드 그룹이 서로 및 다른 네트워크 끝점과 통신하는 방법을 지정
  - NetworkPolicy 리소스는 레이블을 사용하여 포드를 선택
  - 선택한 포드에 허용되는 트래픽을 지정하는 규칙 정의
  - 특정 포드를 선택하는 네임 스페이스에 NetworkPolicy가 있으면 해당 포드는 NetworkPolicy에서 허용하지 않는 연결은 거부
  - Network Policy 사용이 가능한 CNI가 있다.
    - Calico, weavenet 등
  - GCP에서는 따로 설정을 해주어야 한다.
    - 클러스터 만들기 -> 네트워킹 -> 네트워크 정책 사용 체크 필요
    
- Egress
  - 선택된 포드에서 나가는 트레픽에 대한 정책 설정
  - ipBlock을 통해 Block하고자 하는 IP 대역 설정 가능
  - Ports에는 어떤 포드를 막고자 하는지 명시
  - 예시
    ```
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: test-network-policy
      namespace: default
    spec:
      podSelector:
        matchLabels:
          role: db
    policyTypes: # role: db label이 붙은 Pod에 egress 적용
      - Egress
      egress: # whitelist 작성
      - to:
        - ipBlock:
          cidr: 10.0.0.0/24
        ports:
        - protocol: TCP
          port: 5978
    ```
    
- Ingress
  - 선택된 포드로 들어오는 트래픽에 대한 정책 설정
  - IpBlock은 기본적으로 막고자 하는 IP 대역을 설정
  - Except를 사용하여 예외 항목 설정 가능
  - NamespaceSelector와 podSelector를 사용
    - 그룹별 더욱 상세한 정책 설정 가능
  - 예시
    ```
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: test-network-policy
      namespace: default
    spec:
      podSelector:
        matchLabels:
          role: db
      policyTypes:
      - Ingress
      ingress:
      - from:
        - ipBlock: # whitelist ip block
            cidr: 172.17.0.0/16
            except:
            - 172.17.1.0/24
        - namespaceSelector: # whitelist namespace label
            matchLables:
              project: myproject
        - podSelector: # whitelist pod label
            matchLabels:
              role: frontend
        ports:
        - protocol: TCP
          port: 6379
    ```
    - ipBlock, namespace selector, pod selector 에 대한 조건은 모두 or 연산이다.

#### 출처

- DevOps 를 위한 쿠버네티스 마스터 강의
