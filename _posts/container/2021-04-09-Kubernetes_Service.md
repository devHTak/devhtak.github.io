---
layout: post
title: Kubernetes Services
summary: Kubernetes
author: devhtak
date: '2021-04-09 21:41:00 +0900'
category: Container
---

#### POD의 문제점

- POD는 일시적으로 생성한 컨테이너의 집합
- 그렇기 때문에 포드가 지속적으로 생겨났을 때 서비스를 하기에 적합하지 않음
  - IP 주소의 지속적 변동, 로드밸런싱을 관리해줄 또 다른 개체가 필요
  
- 이 문제를 해결하기 위해 서비스라는 리소스가 존재

#### Service 오브젝트

- 포드의 포트를 외부로 노출해 사용자들이 접근하거나, 다른 디플로이먼트의 포드들이 내부적으로 접근하려면 서비스라고 부르는 별도의 쿠버네티스 오브젝트를 생성해야 한다.
- 핵심 기능
  - 여러 개의 포드에 쉽게 접근할 수 있도록 고유한 도메인 이름 부여
  - 여러 개의 포드에 접근할 때, 요청을 분산하는 로드 밸런서 기능 수행
  - 클라우드 플랫폼의 로드 밸런서, 클러스터 노드의 포트등을 통해 포드를 외부로 노출

- 서비스의 종류
  - ClusterIP 타입
    - 쿠버네티스 내부에서만 포드들에 접근할 때 사용. 외부로 포드를 노출하지 않기 때문에 쿠버네티스 클러스터 내부에서만 사용되는 포드에 적합
  - NodePort 타입
    - 포드에 접근할 수 있는 포트를 클러스터의 모든 노드에 동일하게 개방
    - 외부에서 포드에 접근할 수 있다.
    - 접근하는 포트는 랜덤으로 정해지지만, 특정 포트로 접근하도록 설정할 수 있다.
  - LoadBalancer 타입
    - 클라우드 플랫폼에서 제공하는 로드 밸런서를 동적으로 프로비저닝해 포드에 연결
    - NodePort 타입과 마찬가지로 외부에서 포드에 접근할 수 있는 서비스 타입
    - 일반적으로 AWS, GCP와 같은 클라우드 플랫폼 환경에서만 사용 가능

#### Services 요구사항

- 외부 클라이언트가 몇 개이든지 프론트엔드 포드로 연결
- 프론트엔드는 다시 백엔드 데이터베이스로 연결
- 포드의 IP가 변경될 때마다 재설정하지 않도록 해야 한다.

![image](https://user-images.githubusercontent.com/42403023/114118889-8e49b680-9924-11eb-885a-75aade732d7d.png)

** 이미지 출처: DevOps를 위한 Kubernetes 마스터 강의

#### 서비스의 생성 방법

- kubectl의 expose가 가장 쉬운 방법
  ```
  $ kubectl expose deployment hello-world --type=?? --name=my-service --port=??
  ```
  
- YAML을 통해 버전 관리 기능
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: http-go-svc 
  spec:
    selector:
      app: http-go
    ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

  ---

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: http-go
    name: http-go
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: http-go
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: http-go
      spec:
        containers:
        - image: gasbugs/http-go
          name: http-go
          ports:
          - containerPort: 8080
          resources: {}
  status: {}
  ```
    - 80번 포트로 service를 접근하고, 서비스가 POD를 8080 포트로 접근한다.
    - selector의 labels -> service가 POD를 선택할 때 기준이 되는 label
    
  ```
  $ kubectl create -f http-go-svc.yaml
  service/http-go-svc created
  deployment.apps/http-go created
  $ kubectl get all
  NAME                         READY   STATUS    RESTARTS   AGE
  pod/http-go-65b9844c-66hgq   1/1     Running   0          96s

  NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
  service/http-go-svc   ClusterIP   10.97.199.186   <none>        80/TCP    96s
  service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   7m43s

  NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/http-go   1/1     1            1           96s

  NAME                               DESIRED   CURRENT   READY   AGE
  replicaset.apps/http-go-65b9844c   1         1         1       96s
  $ kubectl get pod -o wide
  NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
  http-go-65b9844c-66hgq   1/1     Running   0          110s   172.17.0.6   minikube   <none>           <none>
  ```
  
- 서비스 기능 확인
  - 서비스를 생성하면 EXTERNAL -IP를 아직 받지 못한 것을 확인
    ```
    $ kubectl exec <포드 이름> --curl ip
    ```

#### 포드 간의 통신을 위한 ClusterIP

- 기본적으로 Service type이 CluterIP로 생성된다. 
- 내부에서 로드밸런싱하도록 도와준다. 외부 노출은 하지 않는다.
- 다순의 포드를 하나의 서비스로 묶어서 관리
  - Frontend(Front_POD1, Front_POD2, ..) - Backend_ClusterIP -> Backend(Back_POD1, Back_POD2, ..) - DB_ClusterIP -> DB(DB_POD1, DB_POD2, ..)
    - Backend_ClusterIP
      ```
      apiVersion: v1
      kind: Service
      metadata:
        name: back-end
      spec:
        type: ClusterIP
        ports:
        - targetPort: 80
          port: 80
        selector:
          app: myapp
          type: back-end
      ```
    - DB_ClusterIP
      ```
      apiVersion: v1
      kind: Service
      metadata:
        name: db
      spec:
        type: ClusterIP
        ports: 
        - targetPort: 3006
          port: 3306
        selector:
          app: mysql
          type: db
      ```
    
- 서비스의 세션 고정
  - 서비스가 다수의 포드로 구성하면 웹서비스의 세션이 유지되지 않는다.
    - 여러 POD의 접근했을 때 Session이 유지되지 않으면, 2번 POD에서 로그인한 후 3번 POD로 요청을 보냈는데, Session이 유지되지 않으면 로그인을 확인하지 못한다.
  - 이를 위해 처음 들어왔던 클라이언트 IP를 그대로 유지해주는 방법 필요
  - sessionAffinity: ClientIP라는 옵션을 주면 해결 완료!
    - Session Affinity: 동일한 서버에 요청을 동일 서버로 라우팅하도록 한다.
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: http-go-svc
    spec:
      sessionAffinity: ClientIP
      ports:
      - port: 80
        targetPort: 8080
      selector:
        app: http-go
    ```
  - 접속 확인
    ```
    $ kubectl run -it --rm --image=busybox bash
    / # wget -O- -q 10.97.199.186
    Welcome! http-go-65b9844c-klhj9
    / # wget -O- -q 10.97.199.186
    Welcome! http-go-65b9844c-klhj9
    ```
    - 같은 POD로 접속되는 것을 확인할 수 있다.

- 다중 포트 서비스 방법
  - 포트에 그대로 나열해서 사용
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: http-go-svc
    spec: 
      ports:
      - name: http
        port: 80
        targetPort: 8080
      - name: https
        port: 443
        targetPort: 8443
    ```
      - YAML 작성 법. 복수형(s)가 나오면 그 다음에는 -를 추가하여 list로 구성한다.
    ```
    $ kubectl get svc
    ```
    - http-go-svc에 PORT(S)가 80/TCP, 443/TCP로 여러 PORT(HTTP, HTTPS)가 할당되어 있다.

- 서비스하는 IP 정보 확인
  - 서비스 세부 사항에는 연결될 IP에 대한 정보가 존재
    ```
    $ kubectl describe svc http-go-svc
    Name:              http-go-svc
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=http-go
    Type:              ClusterIP
    IP Families:       <none>
    IP:                10.97.199.186
    IPs:               10.97.199.186
    Port:              <unset>  80/TCP
    TargetPort:        8080/TCP
    Endpoints:         172.17.0.10:8080,172.17.0.6:8080,172.17.0.8:8080
    Session Affinity:  None
    Events:            <none>
    ```
      - Endpoints에 여러 POD의 IP가 들어있고 로드밸런싱할 수 있도록 설정되어 있다.

- 외부 IP 연결 설정 YAML
  - 내부에서 로드밸런싱해서 외부로 나가기 위해 사용
  - 온프레미스에서 클라우드로 이전 작업, 하이브리드 형태에서 사용한다.
  - Service와 Endpoints 리소스 모두 생성 필요
    - external-svc.yaml
      ```
      apiVersion: v1
      kind: Service
      metadata:
        name: external-service
      spec:
        ports:
        - port: 80
      ```
    - external-endpoint.yaml
      ```
      apiVersion: v1
      kind: Endpoints
      metadata:
        name: external-service
      subsets:
        - address:
          - ip: 11.11.11.11
          - ip: 22.22.22.22
          ports:
          - port: 80
      ```
      
  - Service와 Endpoints 연결 구조
    
    ![image](https://user-images.githubusercontent.com/42403023/114120132-e1bd0400-9926-11eb-88a5-896ff6b98ac2.png)
    
    **  이미지 출처: DevOps를 위한 Kubernetes 마스터 강의 
    
#### 서비스 노출하는 세가지 방법

- NodePort: 노드의 자체 포트를 사용하여 포드로 리다이렉션
- LoadBalancer: 외부 게이트웨이를 사용하여 노드 포트로 리다이렉션
- Ingress: 하나의 IP 주소를 통해 여러 서비스를 제공하는 특별 메커니즘

#### NodePort

- Node의 자체 Port를 사용하여 리다이렉션한다.
- NodePort 생성하기
  - Service Yaml 파일 작성
    - type을 NodePort를 지정하여 최종적으로 서비스되는 포트로 지정한다.
    - 30000 ~ 32767 포트만 사용가능
      ```
      apiVersion: v1
      kind: Service
      metadata:
        name: http-go-np
      spec:
        type: NodePort
        selector:
          run: http-go
        ports:
        - port: 80 # 서비스의 포트
          targetPort: 8080 # 포드의 포트
          nodePort: 30001 # 최종적으로 서비스되는 포트
      ```
      ```
      $ kubectl get svc
      NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
      http-go-np   NodePort    10.101.61.80   <none>        80:30001/TCP   7s
      kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        4m13s
      ```
        - External-IP가 none이지만 PORT가 80:30001로 NodePort가 30001로 설정되어 있기 때문에 외부에서 접근할 수 있다.
        - 모든 노드에 동일한 포트가 생성된다.
  
  - GCP 에서 방화벽 해제 후 노드로 직접 접속
    ```
    $ gcloud compute firewall-rules create http-go-svc-rule --allow=tcp:30001
    $ kubectl get node -o wide
    $ curl Node-External-IP:NodePort
    ```

- 노드포트 서비스의 패킷 흐름

  ![image](https://user-images.githubusercontent.com/42403023/114121182-df5ba980-9928-11eb-9983-c64c6eb7e90b.png)
  ** 이미지 출처: https://kubernetes.io/docs/concepts/services-networking/service
  
  - NodePort(3001번 포트)로 Node에 접근 -> API Server(Kubectl Proxy) RR 방식으로 -> POD 접근
    - (참고)RoundRobin: Using Round robin method, client requests are routed to available servers on a cyclical basis. Round robin server load balancing works best when servers have roughly identical computing capabilities and storage capacity.
  - (클러스터 내) Node - Service - POD

- 노드포트를 활용한 로드밸런싱
  
  ![image](https://user-images.githubusercontent.com/42403023/114121224-f39fa680-9928-11eb-8408-1389386b88a8.png)
  ** 이미지 출처: https://en.wikipedia.org/wiki/Kubernetes

#### 로드밸런스

- NodePort 서비스의 확장된 서비스
  - Node 앞에 Load Balancer를 두는 방식
- 클라우드 서비스에서 사용 가능
  - virtual box에서는 안된다.
- yaml 파일에서 타입을 NodePort 대신 LoadBalancer를 설정
- 로드 밸런서의 IP 주소를 통해 서비스에 액세스

- 로드밸런스 생성하기
  - http-go-lb.yaml
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: http-go-lb
    spec:
      type: LoadBalancer
      ports:
      - port: 80 # 서비스의 포트
        targetPort: 8080 # 포드의 포트
      selector:
        app: http-go
    ```
    ```
    $ kubectl get svc
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    http-go-lb   LoadBalancer   10.108.173.114   <pending>     80:31350/TCP   13s
    http-go-np   NodePort       10.100.133.123   <none>        80:30001/TCP   10m
    kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP        10m
    $ curl LoadBalancer_External_IP
    ```
     - http-go-lb의 NodePort를 설정하지 않았지만, 임의의 포트로 설정해준다.

- 로드 밸런스 서비스의 패킷 흐름
  - 클러스터에 로드밸런싱을 해주는 개체 필요
  - 로드밸런스는 노드포트의 기능 포함
  
  ![image](https://user-images.githubusercontent.com/42403023/114121134-c9e67f80-9928-11eb-811b-66958d3f6124.png)
  
  ** 이미지 출처: : https://kubernetes.io/docs/concepts/services-networking/service

#### Ingress

- 인그레스의 필요성
  - 인그레스는 하나의 IP이나 도메인으로 다수의 서비스를 제공
    - L4의 기능으로 NodePort가 제공되어야 사용할 수 있다. 
  - ingress: (어떤 장소에) 들어감, 입장; 들어갈 수 있는 권리, 입장권
  
  ![image](https://user-images.githubusercontent.com/42403023/114121443-55601080-9929-11eb-9668-32e08d9db481.png)
  ** 이미지 출처: DevOps를 위한 Kubernetes 마스터 강의

- 인그레스 생성하기
  - http-go-ingress.yaml
    ```
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: ingress
    spec:
      rules:
      - host: gasbugs.com # 도메인 이름 일치해야 한다. (아닌 경우 404 error 발생)
        http:
          paths:
          - path: / # 연결할 서비스 설정
            backend:
              serviceName: http-go-np
              servicePort: 8080
    ```

- 인그레스 정보 확인 및 접속
  - 접속을 위해서는 반드시 룰에 일치되어야 하므로 HTTP 요청의 host가 gasbugs.com 값으로 가질 수 있도록 설정 (/etc/hosts 변경)
  - 만약 ADDRESS가 장시간 설정되지 않는다면 연결한 노드포트와 포드가 제대로 올라와있는지 확인해야 한다.

  ```
  $ kubectl get ingress
  
  $ sudo vim /etc/hosts
  
  $ curl http:gasbugs.com
  ```
  
- 다수의 서비스 제공
  ```
  apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: ingress
    spec:
      rules:
      - host: gasbugs.com # 도메인 이름 일치해야 한다. (아닌 경우 404 error 발생)
        http:
          paths:
          - path: / # 연결할 서비스 설정
            backend:
              serviceName: http-go-np
              servicePort: 8080
      - host: dict.gasbugs.com
        http:
          paths:
          - path: / # 연결할 서비스 설정
            backend:
              serviceName: http-go-np
              servicePort: 8080
  ```
    - rules에 host 를 list로 하면 다수의 서비스를 제공할 수 있다.

  ```
  $ kubectl get ingress  
  $ sudo vim /etc/hosts
  # hosts 파일을 추가하여 ip <-> domain을 맞춰준다.  
  $ curl www.gasbugs.com  
  $ curl dict.gasbugs.com
  ```
  
- 인그레스 HTTPS 서비스하기
  - TLS 인증성 생성
    ```
    $ openssl genrsa -out tls.key 2048
    $ openssl req -new -x509 -key tls.key -out tls-cert -days 360 -subj /CN=www.gasbugs.com
    $ kubectl create secret tls tls-secret --cret=tls.cert --key=tls.key
    $ curl -k https://www.gasbugs.com
    ```

** 출처: DevOps를 위한 Kubernetes 마스터 강의
