---
layout: post
title: Kubernetes Architecture 및 설치
summary: Kubernetes
author: devhtak
date: '2021-03-30 21:41:00 +0900'
category: Container
---

#### 쿠버네티스 소개

- 쿠버네티스 시작
  - 오랜 세월 동안 구글은 Borg(보그)라는 내부 시스템을 개발
  - 애플리케이션 개발자와 시스템 관리자가 수천 개의 애플리케이션과 서비스를 관리하는 데 도움
  - 조직 규모가 클 때 엄청난 가치를 발휘
  - 수십만 대의 시스템을 가동할 때 사용률이 조금만 향상돼도 수백만 달러의 비용 절감 효과
  - 구글은 보그와 오메가를 15년동안 비밀로 유지하다 2014년 구글 시스템을 통해 얻은 경험을 바탕으로 한 오픈소스 시스템인 '쿠버네티스'를 출시
  - 인프라의 추상화
    - 컨테이너 시스템에서 컨테이너 애플리케이션을 쉽게 배포, 관리하도록 돕는 소프트웨어 시스템
    - 기본 인프라를 추상화해 개발 및 운영팀의 개발, 배포, 관리를 단순화
    - 모든 노드가 하나의 거대한 컴퓨터인 것처럼 수천개의 컴퓨터 노드에서 소프트웨어 애플리케이션을 실행

- 쿠버네티스 장점
  - 애플리케이션 배포 단순화
    - 특정 베어메탈을 필요로 하는 경우 SSD/HHD 등에 라벨링을 하여 배포를 단순화할 수 있다.
    - 가상화 아키텍처에서 크게 두 가지로 분류하는데, 호스트형 가상화와 베어메탈 가상화이다.
    - 호스트형 가상화는 호스트 운영체제가 있고, 그 위에 가상머신을 구현한 방식
    - 베어메탈 가상화는 호스트 운영체제가 없고, 하드웨어 상에 하이퍼바이저가 바로 설치되고, 이 위에 가상 머신을 구현한 방식
  - 하드웨어 활용도 극대화
    - 클러스터의 주변에 자유롭게 이동하여 실행중인 다양한 애플리케이션 구성 요소를 클러스터 노드의 가용 리소스에 최대한 맞춰 서로 섞고 매치
    - 노드의 하드웨어 리소스를 최상으로 활용
  - 상태 확인 및 자가 치유
    - 애플리케이션 구성 요소와 실행되는 노드를 모니터링 하고 노드 장애 발생시 다른 노드로 일정을 자동으로 재조정
    - 자동으로 리소스를 모니터링하고 각 애플리케이션에서 실행되는 인스턴스 수를 계속 조정하도록 지시 가능
  - 오토스케일링
    - 개별 애플리케이션의 부하를 지속적으로 모니터링할 필요 없이 자동으로 리소스를 모니터링하고 각 애플리케이션에서 실행되는 인스턴스 수를 계속 조정하도록 지시 가능
  - 애플리케이션 개발 단순화
    - 버그 발견 및 수정 (완전히 개발환경과 같은 환경 제공)
    - 새로운 버전 출시 시 자동으로 테스트, 이상 발견 시 롤아웃

- 개발자 돕기: 핵심 애플리케이션 기능에 집중
  - 애플리케이션 개발자가 특정 인프라 관련 서비스를 애플리케이션에 구현하지 않아도 된다.
  - 쿠버네티스에 의존하여 서비스 제공
    - 서비스 검색, 확장, 로드밸런싱, 자가 치유, 리더 선출 등
  - 애플리케이션 개발자는 애플리케이션의 실제 기능을 구현하는 데 주력
  - 인프라와 인프라를 통합하는 방법을 파악하는 데 시간을 낭비할 필요가 없다.

- 운영 팀 돕기: 효과적으로 리소스를 활용
  - 실행을 유지하고 서로 통신할 수 있도록 컴포넌트에 정보 제공
  - 애플리케이션이 어떤 노드에서 실행되는지 상관 없다.(신경 쓰지 않아도 된다.)
  - 언제든지 애플리케이션을 재배치 가능
  - 애플리케이션을 혼합하고 매장시킴으로써 리소스를 매칭

#### 쿠버네티스 아키텍처

- 쿠버네티스 클러스터 아키텍처
  - 쿠버네티스의 클러스터는 하드웨어 수준에서 많은 노드로 구성되며 두가지 유형으로 나뉜다.
    - 마스터 노드: 전체 쿠버네티스 시스템을 관리하고 통제하는 쿠버네티스 컨트롤 플레인을 관장한다.
    - 워커 노드: 실제 배포하고자 하는 애플리케이션의 실행을 담당한다.
    - 마스터 노드에 여러 워커 노드를 운영할 수 있다. 마스터 노드 또한 이중화할 수 있다.
    
    ![image](https://user-images.githubusercontent.com/42403023/118449966-a6d09c00-b72e-11eb-8f8f-b9fdbefd1140.png)
    
    - 이미지 출처: https://medium.com/@chkrishna/kubernetes-architecture-f7ca63fff46e  
  
  - Kubernetes Control Plane (Masters)
    - Control Plane에서는 클러스터를 관리하고 클러스터의 기능
    - 단일 마스터 노드에서 실행하거나 여러 노드로 분할되고 복제해 고가용성을 보장
    - 클러스터의 상태를 유지하고 제어하지만 애플리케이션을 실행하지 않는다.
    - 구성 요소
      - kube-apiserver(쿠버네티스 API 서버): 사용자, Control Plain과 통신
      - kube-scheduler(스케줄러): 애플리케이션 예약(애플리케이션의 배포 가능한 각 구성 요소에 워커 노드를 할당)
      - Control-manager: 구성 요소 복제, 워커 노드 추적, 노드 장애 처리 등 클러스터 수준 기능을 실행
      - etcd(데이터 스토리지): 클러스터 구성을 지속적으로 저장하는 안정적인 분산
      
  - Worker Node
    - 컨테이너화된 애플리케이션을 실행하는 시스템
    - 애플리케이션에 서비스를 실행, 모니터링, 제공하는 작업은 다음과 같은 구성요소로 수행
      - Container runtime: 컨테이너를 실행하는 도커
      - Kubelet: APi 서버와 통신하고 노드에서 컨테이너를 관리
      - Kubernetes Service, Proxy: 애플리케이션 간에 네트워크 트래픽을 분산 및 연결

- 쿠버네티스에서 애플리케이션 실행
  - 개발자가 미리 만든 이미지를 이미지 레지스트리에 push한다.
  - 쿠버네티스에 올리길 원하면, yaml 또는 json으로 app descriptor를 작성한다.
  - control plain(master)가 descriptor를 참조하여 worker node의 kublet에 이미지를 배치하도록 명령한다.
  - 이 후 이미지 레지스트리에서 이미지를 worker node에 세팅한 후 실행한다.
  - 개발자는 이미지를 만든 후 push하고 control plain에 descrption을 제출한다.

#### Ubuntu에 쿠버네티스 클러스터 구성

- 쿠버네티스 설치 필요 사항
  - Master 우분투: 쿠버네티스의 마스터 노드가 설정될 호스트
  - Work 노드(option): 필수 사항은 아니지만, 클러스터에 Work 노드 추가 학습
  - 버추얼 박스에서 각 노드에서 복제하면서 반드시 변경해야 할 설정
    - 호스트 이름: /etc/hostname
    - 네트워크 인터페이스 변경
    - NAT 네트워크 설정(NAT랑 다름)
    - (호스트 이름 변경하려면 반드시 리붓)

- 쿠버네티스 우분투에 설치
  - 도커를 먼저 설치
    ```
    $ apt install docker.io
    # 만약 락이 걸려 있는 경우 reboot 후에 하면 된다.
    ```
  - 다음 내용을 install.sh 파일에 작성하고 chmod로 권한을 주고 실행
    ```
    $ gedit install.sh
    sudo apt-get update && sudo apt-get install -y apt-transport-https curl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    # 복사
    # 실행
    $ bash install.sh 
    # 설치 확인
    $ kube + tab
    kubeadm  kubectl  kubelet
    # 설치 완료 후 halt(중단)
    $ halt
    ```
  - 쿠버네티스 설치 사이트
    - https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
  - 신뢰할 수 있는 APT 키 추가
    - apt-get update && apt-get install -y apt-transport-https curl
    - curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

- Kubenrnetes를 관리하는 명령어
  - kubeadm
    - 클러스터를 부트스트랩하는 명령
  - kubelet
    - 클러스터의 모든 시스템에서 실행되는 구성요소로 창 및 컨테이너 시작과 같은 작업을 수행
  - kubectl
    - 커맨드 라인 util은 당신의 클러스터와 대화

- 마스터 노드에서 Work 노드 복사
  - 스냅샷 찍기
    - docker, kubeadm 설치된 스냅샷으로 복원 가능
  - 복제
    - 이름 설정
    - 주소 정책: NAT 네트워크 어댑터 새 MAC 주소 생성
      - MAC 주소를 새로 생성해야 분리하여 사용할 수 있다.
  - 파일 -> 환경설정 -> 네트워크 -> 새로운 NAT 추가
  - 마스터, 워커 노드에 추가한 NAT 추가
    - 이미지 우 클릭 -> 설정 -> 네트워크 -> NAT 네트워크 -> 추가한 NAT 네트워크로 설정
    - 고급, MAC 주소 새로고침
    - 같은 switch에 물릴 수 있도록 해당 설정 진행
  - 확인
    - 세개의 이미지를 올린 후에 ping으로 확인
      ```
      # 세개의 이미지에서 다른 ip 확인
      $ ip addr 
      $ ping 10.0.2.5
      ```
    - 만약 같은 ip로 나온다면 아래 설정을 확인해보자
      - 네트워크 인터페이스의 MAC 주소를 변경하지 않음
      - 네트워크 설정이 NatNetwork로 지정되지 않음

- 마스터 노드, Work 노드 세팅
  - 노드 별로 hostname을 수정해야 한다.
    ```
    $ vi /etc/hostname
    Master
    WorkNode1
    WorkNode2
    # reboot해야 hostname 수정이 가능하다.
    $ reboot
    ```
    
  - 마스터 및 워커 노드 스왑 에러 발생 시 스왑 기능 제거
    ```
    $ sudo swapoff -a // 현재 커널에서 스왑 기능 끄기
    $ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab // 리붓 후에도 스왑 기능 유지
    ```
    - 스왑기능을 비활성 하는 이유
      - 쿠버네티스 1.8 이후, 노드에서 스왑을 비활성해야 한다. (또는, --fail-swap-on을 false로 설정)
      - 쿠버네티스의 아이디어는 인스턴스를 최대한 100%에 가깝게 성능을 발휘하는 것
      - 모든 배포는 CPU/메모리 제한을 고정하는 것이 필요
      - 따라서 스케줄러가 포드를 머신에 보내면 스왑을 사용하지 않는 것이 필요
      - 스왑 발생 시 속도가 느려지는 이슈 발생, 성능을 위한 것
      - 참고 문헌: https://serverfault.com/questions/881517/why-disable-swap-on-kubernetes
      
  - 마스터 노드 초기화 (사용할 포드 네트워크 대역을 설정)
    ```
    $ sudo kubeadm init
    ```
    ```
    ## 초기화 성공시 나오는 메시지
    Your Kubernetes control-plane has initialized successfully!
    To start using your cluster, you need to run the following as a regular user:
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/
    Then you can join any number of worker nodes by running the following on each as root:
      kubeadm join 10.0.2.15:6443 --token dwaoa1.4nf7b81nsfnkxctw \
      --discovery-token-ca-cert-hash
      sha256:b16367e80df58c3dbacfc3961126ae82d68519368d345fdb228addf04cb4ea2f ****
    ```
    - 클러스터를 사용 초기 세팅
      - 마스터 노드
        ```
        ## 다음을 일반 사용자 계정으로 실행 (콘솔에 출력된 메시지를 복붙)
        $ mkdir -p $HOME/.kube
        $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```
      - 워크 노드
        ```
        ## Pod Network 추가
        $ kubeadm join 10.0.2.15:6443 --token clwxjf.xjmjzwgkl364t68n \
        --discovery-token-ca-cert-hash sha256:683486f3864307d970e4c343c2ce29b3f74ed3604720f1acb601d5765826e5c6
        ## 이것을 잘해야 노드 추가 명령어가 잘 실행됩니다!
        ```
      - 마스터 노드 연결 확인
        ```
        $ kubectl get node
        NAME        STATUS     ROLES                  AGE     VERSION
        master      NotReady   control-plane,master   15m     v1.20.5
        worknode1   NotReady   <none>                 8m24s   v1.20.5
        worknode2   NotReady   <none>                 3s      v1.20.5
        ```
- STATUS: Not Ready
  - Network 설정을 해야 사용할 수 있다.
  - 참고문헌: https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
    ```
    # 워커 노드에서 실행
    $ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```
  - node를 확인하면 Ready 상태로 변경된 것을 확인할 수 있다.
    ```
    $ kubectl get nodes
    NAME        STATUS     ROLES                  AGE   VERSION
    master      Ready      control-plane,master   32m   v1.20.5
    worknode1   Ready      <none>                 25m   v1.20.5
    worknode2   Ready      <none>                 17m   v1.20.5
    ```
- 재부팅 후 이슈 해결
  ```
  $ kubectl get nodes
  The connection to the server 10.0.2.15:6443 was refused - did you specify the right host or port?
  ```
  - swap 설정이 제대로 안이뤄져서 그렇다.
    ```
    $ sudo -i
    $ swapoff -a
    $ exit
    $ strace -eopenat kubectl version
    ```
  - swap 설정을 한 후 다시 하면 된다.
 
- 일반적인 사용자와 마스터 노드, 워커 노드 연결관계

  - 실무에서 사용되는 환경
    ![Master and Work](../images/docker/masterandworker.PNG)

  - 현재 설정한 환경
    - kubectl이 쿠버네티스 클러스터 밖에 있어서 제어하는 것이 아닌, master-virtualbox안에 있다.

** 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터
