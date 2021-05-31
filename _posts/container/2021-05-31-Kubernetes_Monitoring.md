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
        
    - cAdvisor, 프로메테우스, EFK
  
  - top으로 볼 수 있는 것은 한계가 있다. (history 저장, UI 등)

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

#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터
