---
layout: post
title: Kubernetes Logging and Monitoring
summary: Kubernetes
author: devhtak
date: '2021-05-31 21:41:00 +0900'
category: Container
---

#### 쿠버네티스 모니터링 시스템과 아키텍처

- 모니터링 서비스 플랫폼
  - 쿠버네티스를 지원하는 다양한 모니터링 플랫폼
    - Heapster(deprecated), Metrics Service
      - 쿠버네티스의 메트릭 수집 모니터링 아키텍처에서 코어메트릭 파이프라인 경량화
      - 힙스터를 deperecated하고 모니터링 표준으로 metics-server 도입
        ```
        $ kubectl top node
        $ kubectl top pod
        ```
      - top으로 볼 수 있는 것은 한계가 있다. (history 저장, UI 등)      
    - cAdvisor, 프로메테우스, EFK
      - 해당 게시글에서는 작성하지 않을 예정으로 추후 추가적인 학습 필요
  
- 리소스 모니터링 도구
  - 쿠버네티스 클러스터 내의 애플리케이션 성능을 검사
  - 쿠버네티스는 각 레벨에서 애플리케이션의 리소스 사용량에 대한 상세 정보를 제공
  - 애플리케이션의 성능을 평가하고 병목 현상을 제거하여 전체 성능 향상을 도모
  - 리소스 메트릭 파이프라인
    - kubectl top 등의 유틸리티 관련된 메트릭들로 제한된 집합을 제공
    - 단기 메모리 저장소인 metrics-server에 의해 수집
    - metrics-server는 모든 노드를 발견하고 kubelet에 CPU와 Memory를 질의
    - kubelet은 kubelet에 통합된 cAdvisor를 통해 레거시 도커와 통합 후 metric-server 리소스 메트릭으로 노출
    - /metrics/resource/v1beta1 API를 사용
  - 완전한 메트릭 파이프라인
    - 보다 풍부한 메트릭에 접근
    - 클러스터의 현재 상태를 기반으로 자동으로 스케일링하거나 클러스터를 조정
    - 모니터링 파이프라인은 kubelet에서 메트릭을 가져옴
    - CNCF 프로젝트인 프로메테우스가 대표적
    - custom.metrics.k7s,io, external.metrics.k8s.io API를 사용
    
- 모니터링 아키텍처
  
![image](https://user-images.githubusercontent.com/42403023/120197724-3eb8a480-c25c-11eb-9fdb-75aa6163ea52.png)

** 이미지 출처: https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md

#### Metrics Server 설치

- Metrics Server는 쿠버네티스에서 리소스 메트릭 파이프라인을 구성하는 가장 기본적인 방법이다.
- Metrics Server가 자동으로 설치되지는 않으므로 직접 설치해야 한다.

- 설치
  - git에서 다운로드 및 설치
    ```
    $ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ```
  - 아직 top 명령을 사용할 수 없다.
    - TLS 통신이 제대로 이뤄지지 않는다.
    - metrics-server deployment 수정 필요
    ```
    $ kubectl edit deploy -n kube-system metrics-server    
    ```

    ![image](https://user-images.githubusercontent.com/42403023/120199991-cc958f00-c25e-11eb-82d1-81538efc5061.png)
      
      - kubelet-insecure-tls
      - kubelet-preferred-address-types=InternalIP
      - 두개 옵션 추가 
      
  - POD 확인
    ```
    server1@server1-VirtualBox:~/metrics-server$ kubectl get pod -n kube-system
    NAME                               READY   STATUS    RESTARTS   AGE
    metrics-server-69f94597d5-6mtdx    0/1     Running   0          18s
    metrics-server-9f459d97b-tzv54     0/1     Running   0          5m4s
    ```
  - top 명령어 확인
    ```
    $ kubectl top node
    NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
    minikube   957m         47%    962Mi           24%     
    ```

#### 애플리케이션 로그관리

- kubernetes 애플리케이션 로그 확인
  - 로그능 컨테이너 단위로 로그 확인 가능
  - 싱글 컨테이너 포드의 경우 포드까지만 지정하여 로그 확인
  - 멀티 컨테이너의 경우 포드 뒤에 컨테이너 이름까지 전달하려 로그 확인
    ```
    $ kubectl logs <pod name> <옵션: container name>
    ```
    
- kubeapi가 정상 동작하지 않는 경우
  - 쿠버네티스에서 돌아가는 리소스들은 모두 docker를 사용
  - 따라서 docker의 로깅 기능 사용
  - docker ps -a 를 사용하여 조회 가능
  - docker logs < container id >를 사용하여 로그 확인 가능

- 예제
  - 쿠버네티스 애플리케이션 로그 확인
    
    ```
    $ kubectl logs kube-proxy-pdqqx -n kube-system
    I0531 12:47:00.961873       1 node.go:172] Successfully retrieved node IP: 192.168.49.2
    I0531 12:47:00.963215       1 server_others.go:142] kube-proxy node IP is an IPv4 address (192.168.49.2), assume IPv4 operation
    W0531 12:47:01.482472       1 server_others.go:578] Unknown proxy mode "", assuming iptables proxy
    I0531 12:47:01.482564       1 server_others.go:185] Using iptables Proxier.
    I0531 12:47:01.489982       1 server.go:650] Version: v1.20.2
    I0531 12:47:01.490803       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_max' to 131072
    I0531 12:47:01.490975       1 conntrack.go:52] Setting nf_conntrack_max to 131072
    E0531 12:47:01.491334       1 conntrack.go:127] sysfs is not writable: {Device:sysfs Path:/sys Type:sysfs Opts:[ro nosuid nodev noexec relatime] Freq:0 Pass:0} (mount options are [ro nosuid nodev noexec relatime])
    I0531 12:47:01.492117       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_tcp_timeout_established' to 86400
    I0531 12:47:01.492530       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_tcp_timeout_close_wait' to 3600
    I0531 12:47:01.516913       1 config.go:315] Starting service config controller
    I0531 12:47:01.517022       1 shared_informer.go:240] Waiting for caches to sync for service config
    I0531 12:47:01.517049       1 config.go:224] Starting endpoint slice config controller
    I0531 12:47:01.517054       1 shared_informer.go:240] Waiting for caches to sync for endpoint slice config
    I0531 12:47:01.627102       1 shared_informer.go:247] Caches are synced for endpoint slice config 
    I0531 12:47:01.629065       1 shared_informer.go:247] Caches are synced for service config
    ```
    
  - 도커를 사용하여 로깅
    ```
    $ docker logs 1dc7c9efa5f9
    [  OK  ] Listening on Docker Socket for the API.
    [  OK  ] Listening on Podman API Socket.
    [  OK  ] Reached target Sockets.
    [  OK  ] Reached target Basic System.
             Starting containerd container runtime...
    [  OK  ] Started D-Bus System Message Bus.
             Starting minikube automount...
             Starting OpenBSD Secure Shell server...
    [  OK  ] Finished minikube automount.
    [  OK  ] Started OpenBSD Secure Shell server.
    [  OK  ] Started containerd container runtime.
             Starting Docker Application Container Engine...
    [  OK  ] Started Docker Application Container Engine.
    [  OK  ] Reached target Multi-User System.
    [  OK  ] Reached target Graphical Interface.
             Starting Update UTMP about System Runlevel Changes...
    [  OK  ] Finished Update UTMP about System Runlevel Changes.
    ```

- 예제
  - kube-apiserver의 로그를 수집하여 /tmp/apiserver.log에 저장하라
  ```
  $ kubectl logs kube-apiserver -n kube-system > /temp/apiserver.log
  ```
    
  - 도커 로그 확인하기
    - /var/log/containers 하위에 도커 로그가 쌓여있다.
    - 파일명을 보면 POD, NS, Container 명 등에 정보로 이름이 구성되어 있다.
    - 파일 내부에 로그가 작성되어 있어 디버깅이 가능하다.

#### Kube 대시보드 설치와 사용

- Kubernetes Dashboard
  - Kubernetes 클러스터 용 범용 웹 기반 UI
  - 사용자는 클러스터에서 실행중인 응용 프로그램을 관리하고 문제를 해결, 클러스터 자체를 관리
  - https://github.com/kubernetes/dashboard

- 설치 방법
  - https://github.com/kubernetes/dashboard
  - 해당 명령어로 git에 올라온 yaml 파일을 바로 적용
    ```
    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
    ```
    - namespace가 kubernetes-dashboard에 있다.
      ```
      $ kubectl get all -n kubernetes-dashboard
      NAME                                             READY   STATUS              RESTARTS   AGE
      pod/dashboard-metrics-scraper-79c5968bdc-r9rdj   0/1     ContainerCreating   0          114s
      pod/kubernetes-dashboard-9f9799597-q6h8c         0/1     ContainerCreating   0          114s

      NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
      service/dashboard-metrics-scraper   ClusterIP   10.100.133.193   <none>        8000/TCP   114s
      service/kubernetes-dashboard        ClusterIP   10.109.60.178    <none>        443/TCP    115s

      NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/dashboard-metrics-scraper   0/1     1            0           114s
      deployment.apps/kubernetes-dashboard        0/1     1            0           114s

      NAME                                                   DESIRED   CURRENT   READY   AGE
      replicaset.apps/dashboard-metrics-scraper-79c5968bdc   1         1         0       114s
      replicaset.apps/kubernetes-dashboard-9f9799597         1         1         0       114s
      ```
    - service/kubernetes-dashboard 가 ClusterIP로 되어 있기 때문에 외부와 통신이 가능한 NodePort로 변경
      ```
      $ kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard
      type: CluterIP -> type: NodePort
      $ kubectl get service kubernetes-dashboard -n kubernetes-dashboard
      NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
      kubernetes-dashboard        NodePort    10.109.60.178    <none>        443:31845/TCP   20m
      ```
    - 이제 http://\<master-ip>:31845
      - master-ip 알아내는 방법
        ```
        $ kubectl cluster-info
        Kubernetes control plane is running at https://192.168.49.2:8443
        KubeDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
        ```
        - https://192.168.49.2:31845 로 접속
        
  - 외부로 443 포트 열고 접속

- 대시보드를 실행하면 인증 화면이 뜬다.

  ![image](https://user-images.githubusercontent.com/42403023/120251991-2c6f5280-c2be-11eb-8407-e2299b084b90.png)
  
- 토큰 사용하기

  ```
  $ kubectl get sa -n kubernetes-dashboard 
  # sa : service account의 약자
  NAME                   SECRETS   AGE
  default                1         24m
  kubernetes-dashboard   1         24m
  $ kubectl get sa -n kubernetes-dashboard -o yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kubernetes-dashboard"}}
    creationTimestamp: "2021-06-01T00:22:46Z"
    labels:
      k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
    resourceVersion: "107883"
    uid: 3f70ca0c-0639-40c2-9035-335f8456acbf
  secrets:
  - name: kubernetes-dashboard-token-5zghh
  # secret에 저장되어 있다
  $ kubectl get secret -n kubernetes-dashboard kubernetes-dashboard-token-5zghh -o yaml
  # 토큰을 확인할 수 있다.
  $ kubectl describe secret -n kubernetes-dashboard kubernetes-dashboard-token-5zghh
  # 토큰을 확인할 수 있다.
  ```

- 권한 부여
  ```
  $ vi kube-dashboard-role-binding.yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: kubernetes-dashboard-rolebinding
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
  $ kubectl create -f kube-dashboard-role-binding.yaml
  ```
    - cluster-admin 권한을 kubernetes-dashboard 에게 주었다.

#### 네트워크 메쉬 환경 Monitoring: Istio

- Istio란?
  - 쿠버네티스는 도커를 편리하게 컨트롤할 수 있게 하는 다양한 이점을 제공한다.
  - 다수의 컨테이너가 동작하는 경우에 각 컨테이너의 트래픽을 관찰하고 정상 동작하는 지 모니터링하기 어렵다.
  - 서비스 메시의 크기와 복잡성이 커짐에 따라 이해하고 관리하기 더 어려워진다.
    - 서비스 메시: 응용 프로그램과 이들 간의 상호 작용을 구성하는 마이크로 서비스 네트워크 구조
    - ex) 로드밸런싱, 장애 복구, 메트릭 및 모니터링
  - Istio는 이런 쿠버네티스 환경의 네트워크 메시 이슈를 보다 간편하게 해결하기 위해 지원하는 환경

- 왜 Istio를 사용하나
  - 쿠버네티스의 복잡성을 감소시킬 수 있다.
  - 서비스의 연결, 보안, 제어 및 관찰 기능을 구현할 수 있다.
  - 해당 기능으로 복잡성을 줄여 개발 팀의 부담을 감소시키고 모든 컨테이너를 로깅하거나 원격 측정, 정책 시스템을 통합할 수 있는 API 플랫폼이다.
  - 서비스의 코드를 거의 변경하지 않고도 로드밸런싱, 서비스 간 인증, 모니터링 등을 통해 배포된 서비스 네트워크를 쉽게 생성 가능하다는 점
  - 네임스페이스에 설정 옵션을 주고 바로 컨테이너를 배포하면 관찰된다.
  - 마이크로서비스 간의 모든 네트워크 통신을 관찰하는 특수 사이드카 프록시를 배치해 각 포드에 istio-proxy 사이드카를 통해 모니터링 할 수 있다.

- 간략 구성 방법
  - Istio는 POD 내의 Proxy를 두어 App이 외부와 통신할 때 Proxy를 통해서 하도록 한다.
  - Proxy는 Istio에게 해당 정보를 알려주고, Istio를 통해 모니터링을 확인할 수 있다.

- Istio 기능

  |기능|설명|
  |---|---|
  |연결|서비스 간의 트래픽 및 API 호출 흐름을 지능적으로 제어하고 다양한 테스트를 수행하며 Red/Black 배포를 통해 점진적으로 업그레이드|
  |보안|관리 인증, 권한 부여 및 서비스 간 통신 암호화를 통해 서비스를 자동으로 보호|
  |제어|정책을 제어하고 실행, 소비자에게 공정하게 분배|
  |관찰|모든 서비스의 풍부한 자동 추적, 모니터링 및 로깅으로 발생 상황 확인|
  
- Istio 설치 및 적용
  - 설치
    - kubectl이 설치되어 있는 main node에 설치
    ```
    $ curl -L https://istio.io/downloadIstio | sh -
    $ cd istio-1.10.0
    $ export PATH=$PWD/bin:$PATH # 환경변수 세팅
    $ istioctl
    Istio configuration command line utility for service operators to
    debug and diagnose their Istio mesh.

    Usage:
      istioctl [command]
    # ... 
    # 설치 확인
    ```
  
  - 프로젝트에 Istio 적용
    - Istio demo 버전 설치
      ```
      # 설치 환경을 확인할 수 있다. demo 버전 설치 예정
      $ istioctl profile list 
      Istio configuration profiles:
        default
        demo
        empty
        external
        minimal
        openshift
        preview
        remote

      # 데모 버전 설치
      $ istioctl manifest apply --set profile=demo
      This will install the Istio 1.10.0 demo profile with ["Istio core" "Istiod" "Ingress gateways" "Egress gateways"] components into the cluster. Proceed? (y/N) y
      ✔ Istio core installed                                                                                   
      ✔ Istiod installed                                                                                       
      ✔ Ingress gateways installed                                                                             
      ✔ Egress gateways installed                                                                              
      ✔ Installation complete
      Thank you for installing Istio 1.10.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/KjkrDnMPByq7akrYAserver1@server1-VirtualBox:~/istio-1.10.0$ 

      # 해당 명령으로 컴포넌트와 애드온을 추가로 설치할 수 있다.
      # 데모를 설치한 경우 모든 것이 설치되었기 때문에 다음 명령이 필요 없다.
      $ istioctl manifest apply --set addonComponents.grafana.enabled=true
      ```
      - profile 별 지원
        
        ![image](https://user-images.githubusercontent.com/42403023/120880119-4634d080-c603-11eb-952b-28720e1c7772.png)
        
        출처: https://istio.io/docs/setup/additional-setup/config-profiles/

  - 샘플 프로젝트에 Istio 적용
    ```
    # default namespace [istio-injection=enabled] 를 labelling 함
    $ kubectl label namespace default istio-injection=enabled
    namespace/bookinfo labeled
    
    # istio에서 제공하는 샘플 프로젝트 실행
    $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    
    # 설치 확인
    $ kubectl get all -n default
    NAME                                  READY   STATUS            RESTARTS   AGE
    pod/details-v1-79f774bdb9-6h5kl       0/2     PodInitializing   0          65s
    pod/productpage-v1-6b746f74dc-d64jg   0/2     PodInitializing   0          63s
    pod/ratings-v1-b6994bb9-5vmzm         0/2     PodInitializing   0          63s
    pod/reviews-v1-545db77b95-87skz       0/2     PodInitializing   0          64s
    pod/reviews-v2-7bf8c9648f-qlppn       0/2     PodInitializing   0          64s
    pod/reviews-v3-84779c7bbc-ff9tt       0/2     PodInitializing   0          64s

    NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    service/details       ClusterIP   10.98.160.44    <none>        9080/TCP   66s
    service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    11d
    service/productpage   ClusterIP   10.96.90.178    <none>        9080/TCP   65s
    service/ratings       ClusterIP   10.103.135.12   <none>        9080/TCP   65s
    service/reviews       ClusterIP   10.98.202.131   <none>        9080/TCP   65s

    NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/details-v1       0/1     1            0           65s
    deployment.apps/productpage-v1   0/1     1            0           64s
    deployment.apps/ratings-v1       0/1     1            0           65s
    deployment.apps/reviews-v1       0/1     1            0           65s
    deployment.apps/reviews-v2       0/1     1            0           65s
    deployment.apps/reviews-v3       0/1     1            0           65s

    NAME                                        DESIRED   CURRENT   READY   AGE
    replicaset.apps/details-v1-79f774bdb9       1         1         0       65s
    replicaset.apps/productpage-v1-6b746f74dc   1         1         0       64s
    replicaset.apps/ratings-v1-b6994bb9         1         1         0       65s
    replicaset.apps/reviews-v1-545db77b95       1         1         0       65s
    replicaset.apps/reviews-v2-7bf8c9648f       1         1         0       65s
    replicaset.apps/reviews-v3-84779c7bbc       1         1         0       65s
    ```

- 게이트웨이 설치와 관찰
  - 애플리케이션을 생성했지만 게이트웨이를 추가로 생성해야 외부로 접근이 가능해진다.
  - gateway는 istio가 배포한 커스텀된 오브젝트이다.
  - 마치 ingress처럼 동작한다.
  
  - 설치  
    ```
    # 게이트웨이 설치
    $ kubectp apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
    gateway.networking.istio.io/bookinfo-gateway created
    virtualservice.networking.istio.io/bookinfo created
    
    # 게이트웨이 조회
    $ kubectl get gateway
    NAME               AGE
    bookinfo-gateway   18s
    ```
  
  - gateway 확인
    ```
    $ vi samples/bookinfo/networking/bookinfo-gateway.yaml
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: bookinfo-gateway
    spec:
      selector:
        istio: ingressgateway # use istio default controller
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "*"
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: bookinfo
    spec:
      hosts:
      - "*"
      gateways:
      - bookinfo-gateway
      http:
      - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
        route:
        - destination:
            host: productpage
            port:
              number: 9080
    
    # istio-system namespace에 셀렉터로 연결된 istio=ingressgateway svc를 찾아보면 istio 디폴트 컨트롤러를 확인할 수 있다.
    # 클라우드 환경이면 EXTERNAL-IP가 pending이 아닌 IP 정보가 설정된다.
    $ kubectl get svc -n istio-system -l istio=ingressgateway --all-namespaces
    NAMESPACE      NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
    istio-system   istio-ingressgateway   LoadBalancer   10.111.61.240   <pending>
    PORT(S)                                                                     AGE
    15021:31141/TCP,80:31777/TCP,443:30517/TCP,31400:31292/TCP,15443:30989/TCP   14m
    ```
    - 127.0.0.1:31777/productpage 로 접근 확인할 수 있다.

- 연결 구조
  ```
  1. 외부 트래픽
  2. name:istio-ingressgateway(kind: service)
  3. name:istio-ingressgateway-xxxxx(kind: pod)
  4. name: bookinfo-gateway(kind: Gateway)
  5. name: bookinfo (kind: VirtualService)
  6. name: productpage (kind: service)
  7. name: productpage-xxxxx (kind-pod)
  ```
  
- 예제 프로젝트: bookinfo.yaml
  - 구성 아키텍처

    ![image](https://user-images.githubusercontent.com/42403023/120880145-649acc00-c603-11eb-921a-ebd31ab6c1a2.png) 
    
    출처: https://istio.io/docs/examples/bookinfo/

  ```
  # Copyright Istio Authors
  #
  #   Licensed under the Apache License, Version 2.0 (the "License");
  #   you may not use this file except in compliance with the License.
  #   You may obtain a copy of the License at
  #
  #       http://www.apache.org/licenses/LICENSE-2.0
  #
  #   Unless required by applicable law or agreed to in writing, software
  #   distributed under the License is distributed on an "AS IS" BASIS,
  #   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  #   See the License for the specific language governing permissions and
  #   limitations under the License.

  ##################################################################################################
  # This file defines the services, service accounts, and deployments for the Bookinfo sample.
  #
  # To apply all 4 Bookinfo services, their corresponding service accounts, and deployments:
  #
  #   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
  #
  # Alternatively, you can deploy any resource separately:
  #
  #   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l service=reviews # reviews Service
  #   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l account=reviews # reviews ServiceAccount
  #   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l app=reviews,version=v3 # reviews-v3 Deployment
  ##################################################################################################

  ##################################################################################################
  # Details service
  ##################################################################################################
  apiVersion: v1
  kind: Service
  metadata:
    name: details
    labels:
      app: details
      service: details
  spec:
    ports:
    - port: 9080
      name: http
    selector:
      app: details
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: bookinfo-details
    labels:
      account: details
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: details-v1
    labels:
      app: details
      version: v1
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: details
        version: v1
    template:
      metadata:
        labels:
          app: details
          version: v1
      spec:
        serviceAccountName: bookinfo-details
        containers:
        - name: details
          image: docker.io/istio/examples-bookinfo-details-v1:1.16.2
          imagePullPolicy: IfNotPresent
          ports:
          - containerPort: 9080
          securityContext:
            runAsUser: 1000
  ---
  ##################################################################################################
  # Ratings service
  ##################################################################################################
  apiVersion: v1
  kind: Service
  metadata:
    name: ratings
    labels:
      app: ratings
      service: ratings
  spec:
    ports:
    - port: 9080
      name: http
    selector:
      app: ratings
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: bookinfo-ratings
    labels:
      account: ratings
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ratings-v1
    labels:
      app: ratings
      version: v1
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: ratings
        version: v1
    template:
      metadata:
        labels:
          app: ratings
          version: v1
      spec:
        serviceAccountName: bookinfo-ratings
        containers:
        - name: ratings
          image: docker.io/istio/examples-bookinfo-ratings-v1:1.16.2
          imagePullPolicy: IfNotPresent
          ports:
          - containerPort: 9080
          securityContext:
            runAsUser: 1000
  ---
  ##################################################################################################
  # Reviews service
  ##################################################################################################
  apiVersion: v1
  kind: Service
  metadata:
    name: reviews
    labels:
      app: reviews
      service: reviews
  spec:
    ports:
    - port: 9080
      name: http
    selector:
      app: reviews
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: bookinfo-reviews
    labels:
      account: reviews
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: reviews-v1
    labels:
      app: reviews
      version: v1
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: reviews
        version: v1
    template:
      metadata:
        labels:
          app: reviews
          version: v1
      spec:
        serviceAccountName: bookinfo-reviews
        containers:
        - name: reviews
          image: docker.io/istio/examples-bookinfo-reviews-v1:1.16.2
          imagePullPolicy: IfNotPresent
          env:
          - name: LOG_DIR
            value: "/tmp/logs"
          ports:
          - containerPort: 9080
          volumeMounts:
          - name: tmp
            mountPath: /tmp
          - name: wlp-output
            mountPath: /opt/ibm/wlp/output
          securityContext:
            runAsUser: 1000
        volumes:
        - name: wlp-output
          emptyDir: {}
        - name: tmp
          emptyDir: {}
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: reviews-v2
    labels:
      app: reviews
      version: v2
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: reviews
        version: v2
    template:
      metadata:
        labels:
          app: reviews
          version: v2
      spec:
        serviceAccountName: bookinfo-reviews
        containers:
        - name: reviews
          image: docker.io/istio/examples-bookinfo-reviews-v2:1.16.2
          imagePullPolicy: IfNotPresent
          env:
          - name: LOG_DIR
            value: "/tmp/logs"
          ports:
          - containerPort: 9080
          volumeMounts:
          - name: tmp
            mountPath: /tmp
          - name: wlp-output
            mountPath: /opt/ibm/wlp/output
          securityContext:
            runAsUser: 1000
        volumes:
        - name: wlp-output
          emptyDir: {}
        - name: tmp
          emptyDir: {}
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: reviews-v3
    labels:
      app: reviews
      version: v3
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: reviews
        version: v3
    template:
      metadata:
        labels:
          app: reviews
          version: v3
      spec:
        serviceAccountName: bookinfo-reviews
        containers:
        - name: reviews
          image: docker.io/istio/examples-bookinfo-reviews-v3:1.16.2
          imagePullPolicy: IfNotPresent
          env:
          - name: LOG_DIR
            value: "/tmp/logs"
          ports:
          - containerPort: 9080
          volumeMounts:
          - name: tmp
            mountPath: /tmp
          - name: wlp-output
            mountPath: /opt/ibm/wlp/output
          securityContext:
            runAsUser: 1000
        volumes:
        - name: wlp-output
          emptyDir: {}
        - name: tmp
          emptyDir: {}
  ---
  ##################################################################################################
  # Productpage services
  ##################################################################################################
  apiVersion: v1
  kind: Service
  metadata:
    name: productpage
    labels:
      app: productpage
      service: productpage
  spec:
    ports:
    - port: 9080
      name: http
    selector:
      app: productpage
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: bookinfo-productpage
    labels:
      account: productpage
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: productpage-v1
    labels:
      app: productpage
      version: v1
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: productpage
        version: v1
    template:
      metadata:
        labels:
          app: productpage
          version: v1
      spec:
        serviceAccountName: bookinfo-productpage
        containers:
        - name: productpage
          image: docker.io/istio/examples-bookinfo-productpage-v1:1.16.2
          imagePullPolicy: IfNotPresent
          ports:
          - containerPort: 9080
          volumeMounts:
          - name: tmp
            mountPath: /tmp
          securityContext:
            runAsUser: 1000
        volumes:
        - name: tmp
          emptyDir: {}
  ---
  
- Kiali를 활용하여 Istio 대시보드
  
  - istioctl의 dashboard 명령을 사용하면 다양한 서비스에 접근할 수 있다
    ```
    $ istioctl dashboard 
    Access to Istio web UIs

    Usage:
      istioctl dashboard [flags]
      istioctl dashboard [command]

    Aliases:
      dashboard, dash, d

    Available Commands:
      controlz    Open ControlZ web UI
      envoy       Open Envoy admin web UI
      grafana     Open Grafana web UI
      jaeger      Open Jaeger web UI
      kiali       Open Kiali web UI
      prometheus  Open Prometheus web UI
      zipkin      Open Zipkin web UI
    ```

  - Kiali 명령으로 확인
    ```
    $ istioctl dashboard kiali
    # 기본 계정: admin/admin
    ```
    
#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터
