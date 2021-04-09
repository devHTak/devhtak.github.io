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

#### Services 요구사항

- 외부 클라이언트가 몇 개이든지 프론트엔드 포드로 연결
- 프론트엔드는 다시 백엔드 데이터베이스로 연결
- 포드의 IP가 변경될 때마다 재설정하지 않도록 해야 한다.

![image](https://user-images.githubusercontent.com/42403023/114118889-8e49b680-9924-11eb-885a-75aade732d7d.png)
** 이미지 출처: DevOps를 위한 Kubernetes 마스터 강의

#### 서비스의 생성 방법

- kubectl의 expose가 가장 쉬운 방법
  
- YAML을 통해 버전 관리 기능
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: http-go-svc
  spec:
    ports:
    - port: 80
      targetPort: 8080
    selector:
      app: http-go
  ```
    - 80번 포트로 service를 접근하고, 서비스가 POD를 8080 포트로 접근한다.
    - selector의 labels -> service가 POD를 선택할 때 기준이 되는 label
    
  ```
  $ kubectl create -f http-go-svc.yaml
  
  $ kubectl create -f http-go-rs.yaml
  
  $ kubectl get svc
  ```
  
- 서비스 기능 확인
  - 서비스를 생성하면 EXTERNAL -IP를 아직 받지 못한 것을 확인
    ```
    $ kubectl exec <포드 이름> --curl ip
    ```

#### 포드 간의 통신을 위한 ClusterIP

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
  - 이를 위해 처음 들어왔던 클라이언트 IP를 그대로 유지해주는 방법 필요
  - sessionAffinity: ClientIP라는 옵션을 주면 해결 완료!
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
    ```
    $ kubectl get svc
    ```
    - http-go-svc에 PORT(S)가 80/TCP, 443/TCP로 여러 PORT(HTTP, HTTPS)가 할당되어 있다.

- 서비스하는 IP 정보 확인
  - 서비스 세부 사항에는 연결될 IP에 대한 정보가 존재
    ```
    $ kubectl describe svc http-go-svc
    ```

- 외부 IP 연결 설정 YAML
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
    ** 이미지 출처: DevOps를 위한 Kubernetes 마스터 강의
    
#### 서비스 노출하는 세가지 방법

- NodePort: 노드의 자체 포트를 사용하여 포드로 리다이렉션
- LoadBalancer: 외부 게이트웨이를 사용하여 노드 포트로 리다이렉션
- Ingress: 하나의 IP 주소를 통해 여러 서비스를 제공하는 특별 메커니즘

#### NodePort

- NodePort 생성하기
  - Service Yaml 파일 작성
    - type을 NodePort를 지정
    - 30000 ~ 32767 포트만 사용가능
      ```
      apiVersion: v1
      kind: Service
      metadata:
        name: http-go-svc
      spec:
        type: NodePort
        ports:
        - port: 80 # 서비스의 포트
          target: 8080 # 포드의 포트
          nodePort: 30001 # 최종적으로 서비스되는 포트
        selector
          app: http-go
      ```
  
  - GCP 에서 방화벽 해제 후 노드로 직접 접속
    ```
    $ gcloud compute firewall-rules create http-go-svc-rule --allow=tcp:30001
    $ kubectl get node -o wide
    $ curl IP
    ```

- 노드포트 서비스의 패킷 흐름

  ![image](https://user-images.githubusercontent.com/42403023/114121182-df5ba980-9928-11eb-9983-c64c6eb7e90b.png)
  ** 이미지 출처: https://kubernetes.io/docs/concepts/services-networking/service

- 노드포트를 활용한 로드밸런싱
  
  ![image](https://user-images.githubusercontent.com/42403023/114121224-f39fa680-9928-11eb-8408-1389386b88a8.png)
  ** 이미지 출처: https://en.wikipedia.org/wiki/Kubernetes

#### 로드밸런스

- NodePort 서비스의 확장된 서비스
- 클라우드 서비스에서 사용 가능
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
    $ curl IP
    ```    

- 로드 밸런스 서비스의 패킷 흐름
  - 클러스터에 로드밸런싱을 해주는 개체 필요
  - 로드밸런스는 노드포트의 기능 포함
  
  ![image](https://user-images.githubusercontent.com/42403023/114121134-c9e67f80-9928-11eb-811b-66958d3f6124.png)
  
  ** 이미지 출처: : https://kubernetes.io/docs/concepts/services-networking/service

#### Ingress

- 인그레스의 필요성
  - 인그레스는 하나의 IP이나 도메인으로 다수의 서비스를 제공
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
    - host 반복

  ```
  $ kubectl get ingress
  
  $ sudo vim /etc/hosts
  
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
