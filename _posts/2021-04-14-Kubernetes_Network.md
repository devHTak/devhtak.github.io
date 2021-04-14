---
layout: post
title: Kubernetes Netowrk
summary: Kubernetes
author: devhtak
date: '2021-04-09 21:41:00 +0900'
category: Container
---

#### Kubernetes Netowrk Model

- 한 포드에 있는 다수의 컨테이너끼리 통신
- 포드끼리 통신
- 포드와 서비스 사이의 통신
- 외부 클라이언트와 서비스 사이의 통신

- 실습 전 설치 필요
  ```
  $ sudo apt install net-tools
  ```
  
#### 한 포드에 있는 다수의 컨테이너끼리 통신

- pause 명령을 실행하여 아무 동작을 하지 않는 빈 컨테이너 생성
- 인터페이스 공유
- 포트를 겹치게 구성하지 못하는 것이 특징
  - 도커의 네트워크 구조
    - eth0 (10.0.2.4) -> docker0 (127.17.0.1) -> veth0(172.17.0.2 - container1), veth1(172.17.0.3 - container2)

  - 컨테이너 간의 인터페이스를 공유하는 구조
    - eth0 (10.0.2.4) -> docker0 (172.17.0.1) -> veth0 (172.17.0.2 - pause, container1, container2)
    - pause란 사이드 컨테이너는 컨테이너 공유할 수 있도록 interface를 만들고 공유한다.
  
- 도커의 기능을 사용해 쿠버네티스 컨테이너를 관찰
- 각 포드마나 하나의 puase 이미지 실행
  ```
  $ sudo docker ps | grep pause
  ```
  
#### 포드끼리 통신

- 포드끼리의 통신을 위해서는 CNI 플러그인이 필요
- ACI, AOS, AWS VPN CNI, CNI-Genie, GCE, flannerl, Weave Net, Calico 등
- weavenet 네트워크 모델
  ![image](https://user-images.githubusercontent.com/42403023/114671694-55a04780-9d3f-11eb-802b-6b4ad41dac89.png)
  - 이미지 출처: https://www.objectif-libre.com/en/blog/2018/07/05/k8s-network-solutions-comparison/

- CNI 플러그인 지원 기능 설명
  
  |기능|설명|
  |---|---|
  |Network Model|VXLAN: 캡슐화된 네트워킹으로 이론적으로 속도가 느림 / Layer2 NW: 캡슐화되지 않았기 때문에 오버헤드의 영향이 없음|
  |RouteDistribution|BGP 프로토콜 사용, 네트워크 세그먼트에 분할된 클러스터를 구축하려는 경우 사용, 인터넷에서 라우팅 및 연결 가능성 정보를 교환하도록 설계된 외부 게이트웨이 프로토콜|
  |Network Policies|네트워크 정책을 사용하여 포드가 서로 통신할 수 있는 규칙 적용|
  |Mesh Networking|쿠버네티스 클러스터 간에 "Pod to Pod" 네트워킹이 가능, Kubernetes 페레이션이 아닌 포드간의 순수한 네트워킹|
  |Encryption|네트워크 컨트롤 플레인을 암호화하여 모든 TCP 및 UDP 트래픽을 암호화|
  |Ingress /Egress Policies|들어오거나 나가는 패킷에 대한 통신 제어|
  
- 프로세스 및 포트 확인
  ```
  $ sudo netstat -antp | grep weave
  $ sudo docker ps | grep weaver
  $ ps -eaf | grep 19979
  ```

#### 포드와 서비스 사이의 통신

- ClusterIP를 생성하면 iptables의 설정 적용
- kube-proxy라는 컴포넌트로 서비스 트래픽 제어
- iptables는 리눅스 커널 기능인 netfilter를 사용하여 트래픽 제어
  ![image](https://user-images.githubusercontent.com/42403023/114673719-749fd900-9d41-11eb-80c8-f71322b79c65.png) 
  - 이미지 출처: https://ko.wikipedia.org/wiki/Iptables

- 서비스 IP를 통해 10.3.241.152를 요청하는 흐름을 나타낸다.
  ![image](https://user-images.githubusercontent.com/42403023/114673832-95682e80-9d41-11eb-929d-73a356fd2857.png)
  - 이미지 출처: https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82

- ClusterIP를 생성하면 iptables의 설정 적용이 된다.
- iptable에서 목록을 확인하는 실습
  ```
  $ kubectl get svc --all-namespaces
  NAMESPACE     NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
  default       http-go-svc   NodePort    10.109.167.85   <none>        80:32763/TCP             107m
  default       kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP                  108m
  kube-system   kube-dns      ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   13d
  $ sudo iptables -S -t nat | grep 10.96
  ```

#### 외부 클라이언트와 서비스 사이의 통신

- netfilter와 kube-proxy 기능을 사용해 원하는 서비스 및 포드로 연결
  ![image](https://user-images.githubusercontent.com/42403023/114674185-eed05d80-9d41-11eb-8a8a-ed9a811f64d5.png)
  - 이미지 출처: https://medium.com/google-cloud/understandingkubernetes-networking-ingress-1bc341c84078

#### DNS 서비스를 이용한 서비스 검색

- 서비스를 생성하면 대응되는 DNS 엔트리 생성
- 해당 엔트리는 <서비스-이름>.<네임스페이스-이름>.svc.cluster.local의 형식을 갖는다
  ```
  $ kubectl exec -it http-go-rs-4152m bash
  # curl http-go-svc.default.svc.cluster.local
  # curl http-go-svc.default
  # curl http-go-svc
  ```

#### CoreDNS

- CoreDNS가 DNS를 구성하고 있다. 내부에서 DNS 서버 역할을 하는 POD이 존재
- 각 미들웨어를 통해 로깅, 캐시, 쿠버네티스를 질의하는 등의 기능을 갖는다.
  ![image](https://user-images.githubusercontent.com/42403023/114681284-f47d7180-9d48-11eb-85f5-9a990197872b.png)
  - 이미지 출처: https://weekly-geekly.github.io/articles/331872/index.html

- 해당 DNS에는 configmap 저장소를 사용하여 설정 파일 컨트롤
- Corefile을 통해 현재 클러스터의 NS를 지정
  ```
  $ kubectl get configmap coredns -n kube-system -o yaml
  apiVersion: v1
  data:
    Corefile: |
      .:53 {
          errors
          health {
             lameduck 5s
          }
          ready
          kubernetes cluster.local in-addr.arpa ip6.arpa {
             pods insecure
             fallthrough in-addr.arpa ip6.arpa
             ttl 30
          }
          prometheus :9153
          forward . /etc/resolv.conf {
             max_concurrent 1000
          }
          cache 30
          loop
          reload
          loadbalance
      }
  kind: ConfigMap
  metadata:
    creationTimestamp: "2021-04-01T08:02:27Z"
    managedFields:
    - apiVersion: v1
      fieldsType: FieldsV1
      fieldsV1:
        f:data:
          .: {}
          f:Corefile: {}
      manager: kubeadm
      operation: Update
      time: "2021-04-01T08:02:27Z"
    name: coredns
    namespace: kube-system
    resourceVersion: "263"
    uid: d9890596-b6cd-4fa8-8463-4fdc31018a5f
  ```
  
- POD에서도 Subdomain을 사용하면 DNS 서비스 사용 가능
  - yaml 파일의 호스트 이름은 포드의 metadata.name을 따른다.
  - 필요한 경우 hostname을 따로 선택 가능
  - subdomain은 서브도메인을 지정하는데 사용
  - 서브 도메인을 설정하면 FQDN 사용 가능
    - FQDN
      ```
      <hostname>.<subdomain>.<namespace>.svc.clustre-domain.example
      busybox-1.default-subdomain.default.svc.cluster-domain.example
      ```
#### 연습문제

- 네임스페이스 blue에 jenkins 이미지를 사용하는 pod-jenkins 디플로이먼트를 생성하고 이를 위한 서비스 srv-jenkins를 생성하라.
  ```
  $ kubectl create ns blue --dry-run=client -o yaml > blue-jenkins-svc-deploy.yaml
  $ kubectl run pod-jenkins --image=jenkins --port=8080 --dry-run=client -n blue >> blue-jenkins-svc-deploy.yaml
  # blue라는 namespace에 jenkins 이미지 실행
  $ vi blue-jenkins-svc-deploy.yaml 
  apiVersion: v1
  kind: Namespace
  metadata:
    creationTimestamp: null
    name: blue    
  spec: {}
  status: {}
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      run: pod-jenkins
    name: pod-jenkins
    namespace: blue
  spec:
    replicas: 1
    selector:
      matchLabels:
        run: pod-jenkins
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          run: pod-jenkins
      spec:
        containers:
        - image: jenkins
          name: pod-jenkins
          ports:
          - containerPort: 8080
          resources: {}
  status: {}
  $  kubectl create -f blue-jenkins-svc-deploy.yaml 
  namespace/blue created
  deployment.apps/pod-jenkins created
  $ kubectl get deploy --namespace blue
  NAME          READY   UP-TO-DATE   AVAILABLE   AGE
  pod-jenkins   0/1     1            0           2m20s
  $ kubectl expose deploy pod-jenkins --name srv-jenkins --namespace blue --dry-run=client -o yaml >> blue-jenkins-svc-deploy.yaml
  # 서비스 생성 하위 추가
  $ vi blue-jenkins-svc-deploy
  ---
  apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: null
    labels:
      run: pod-jenkins
    name: svc-jenkins
    namespace: blue
  spec:
    ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
    selector:
      run: pod-jenkins
  status:
    loadBalancer: {}
  $ kubectl create -f blue-jenkins-svc-deploy.yaml 
  service/svc-jenkins created
  Error from server (AlreadyExists): error when creating "blue-jenkins-svc-deploy.yaml": namespaces "blue" already exists
  Error from server (AlreadyExists): error when creating "blue-jenkins-svc-deploy.yaml": deployments.apps "pod-jenkins" already exists
  ```
- default 네임스페이스의 http-go 이미지의 curl을 사용하여 pod-jenkis:8080을 요청하라.
  ```
  $ kubectl run http-go --image=gasbugs/http-go
  pod/http-go created
  $ kubectl get pod
  NAME                     READY   STATUS              RESTARTS   AGE
  http-go                  0/1     ContainerCreating   0          3s
  ```
  
- 연결 확인
  ```
  $ kubectl exec http-go -- curl srv-jenkins.blue:8080
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0curl: (6) Could not resolve host: srv-jenkins.blue
  command terminated with exit code 6
  ```

** 데브옵스(DevOps)를 위한 쿠버네티스 마스터
