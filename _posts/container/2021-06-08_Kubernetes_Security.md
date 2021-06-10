---
layout: post
title: Kubernetes Cluster Maintaining and security
summary: Kubernetes
author: devhtak
date: '2021-06-08 21:41:00 +0900'
category: Container
---

#### Node OS 업그레이드 절차

- 노드의 유지 보수
  - 커널 업그레이드, libc 업그레이드, 하드웨어 복구 등과 같이 노드를 재부팅해야 하는 경우
  - 노드를 갑자기 내리는 경우 다양한 문제가 발생할 수 있다.
  - 포드는 노드가 내려가서 5분(default) 이상 돌아오지 않으면 그 때 다른 노드에 포드를 복제
  - 업그레이드 프로세스를 보다 효율적으로 제어하려면 하나의 노드 씩 다음 절차를 진행
    - drain node -> update node -> uncordon node -> drain node -> ...(circular 순환)
    
- 예시
  - 현재 노드 확인
    ```
    $ kubectl get node
    NAME                         STATUS   ROLES                  AGE   VERSION
    gke-standard-cluster-1-...   Ready    control-plane,master   68d   v1.20.2
    ```
  - 노드 drain(배수)
    ```
    $ kubectl drain gke-standard-cluster-1-...
    ```
    - 노드에 스케줄링된 포드들은 모두 내리는 작업
    - 노드를 내리는 작업에서 문제가 발생하는 경우 권장하는 옵션을 사용
  - 옵션을 모두 사용하여 다시 시도
    ```
    $ kubectl drain gke-standard-cluster-1-... --delete-local-data --ignore-daemonsets --force
    ```
  - 노드 다시 확인
    ```
    $ kubectl get node
    NAME                         STATUS                     ROLES                  AGE   VERSION
    gke-standard-cluster-1-...   Ready,SchedulingDisabled   control-plane,master   68d   v1.20.2
    ```
    - SchedulingDisabled 상태로 변경
    ```
    $ kubectl get pod -o wide | grep dlzj #node 확인
    출력 결과 없음
    ```
  - OS 업그레이드 후 노드 복구 (uncordon: 저지선을 철거하다)
    ```
    $ kubectl uncordon gke-standard-cluster-1-...
    gke-standard-cluster-1-... uncordon
    $ kubectl get node
    NAME                        STATUS   ROLES                  AGE   VERSION
    gke-standard-cluster-1-...  Ready    control-plane,master   68d   v1.20.2
    ```

  - cordon 과 drain 차이
    - drain은 스케줄링된 모든 포드를 다른 노드로 리스케줄링을 시도하고 저지선을 만들어 더 이상 노드에 포드가 만들어지지 않도록 한다.
    - 반변 cordon은 저지선만 설치하는 것으로 현재 갖고 있는 포드를 그대로 유지하면서 새로운 포드에 대해서만 스케줄링을 거부한다.

#### 쿠버네티스 버전 업데이트

- 쿠버네티스 버전
  - 2014년 처음 v0.4를 시작하여 2015년에 v1.0으로 메이저 버전 업그레이드
  - 버전이 나타내는 내용
    - MAJOR. MINOR(특징, 기능). PATCH(버그 픽스)
    - v1.14.1 <-> MAJOR.MINOR.PATCH
  - 버그를 고치기 위해 alpha와 beta로 나누어 출시 후 정식버전 진행
    - alpha: 추가되는 기능들을 disable 한 형태로 배포
    - beta: 추가 기능들을 enable한 형태로 배포
    - release: 안정화된 버전을 배포
  - 패키지마다 모든 컴포넌트는 동일한 버전으로 배포
    - API Server, Controller Manager, Scheduler, Kubelet, Kube Proxy 등
    - ETCD, Network, CoreDNS는 제외

- 쿠버네티스에서 버전 호환
  - 쿠버네티스는 apiserver를 기준으로 호환성을 제공
    
    |애플리케이션|지원 버전 범위(apiserver minor version)|예시(1.13 버전)|
    |---|---|---|
    |kube-apiserver|x ~ x-1|1.13, 1.12|
    |kubelet|x ~ x-2|1.13, 1.12, 1.11|
    |kube-controller-manager, kube-scheduler, cloud-controller-manager|x ~ x-2|1.13, 1.12,|
    |kubectl|x ~ x-2|1.14, 1.13, 1.12|

- 쿠버네티스 버전 업그레이드가 필요한 경우
  - 노드의 OS 업데이트와 하나씩 drain 하며 버전을 업데이트 하는 방법 사용 (롤링 업데이트)
  - 업데이트를 적용할 때는 마이너 버전이 하나씩 업데이트 되도록 설정하는 것이 좋음
    ```
    master $ kubectl drain node-1
    master $ ssh node-1
    
    node-1 $ apt-get upgrade -y kubeadm=1.12.0-00 # 1.11 버전에서 업데이트 예시
    node-1 $ apt-get upgrade -y kubelet=1.12.0-00 # 1.11 버전에서 업데이트 예시
    node-1 $ kubeadm upgrade node config --kubelet-version v1.12.0
    node-1 $ systemctl restart kubelet
    node-1 $ exit
    
    master $ kubectl uncordon node-1
    ```

#### 백업과 복원

- 백업 리소스
  - 포드의 정보 파일 YAML
    ```
    $ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
    $ kubectl create -f all-deploy-services.yaml
    ```
    
  - ETCD 데이터 베이스
    ```
    $ $ sudo ETCDCTL_API=3 ./etcdctl --endpoints=127.0.0.1:2379 \
      --cacert /etc/kubernetes/pki/etcd/ca.crt \
      --cert /etc/kubernetes/pki/etcd/server.crt \
      --key /etc/kubernetes/pki/etcd/server.key \
      snapshot save snapshotdb
    $ etcdctl --write-out=table snapshot status snapshotdb
    ```
    - 생겨난 all-deploy-services.yaml 파일과 스냅샷을 뜬 snapshotdb를 복원하고자 하는 위치로 이동시킨다.
    
  - Persistent Volume: 일반적인 방법으로 백업

- 복원
  - Etcd 백업 파일 복원 명령
    ```
    $ sudo ETCDCTL_API=3 ./etcdctl --endpoints=127.0.0.1:2379 \
      --cacert /etc/kubernetes/pki/etcd/ca.crt \
      --cert /etc/kubernetes/pki/etcd/server.crt \
      --key /etc/kubernetes/pki/etcd/server.key \
      --data-dir /var/lib/etcd-restore \
      --name master \
      --initial-cluster-token this-is-token \
      --initial-advertise-peer-urls https://127.0.0.1:2738 \
      snapshot restore ~/yaml/snapshotdb
    ```
    - 옵션 정보: https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/configuration.md
    - cacert, cert, key는 HTTPS를 위하여 설정하였다
    - restore 확인
      - msg에 restore 완료라는 문구가 뜬다.
      - /var/lib/etcd-restore/member가 존재하면 성공
      
  - etcd.yml 스태틱 포드 수정
    ```
    $ sudo vim /etc/kubernetes/manifest/etcd.yaml
    ```
    - 다음 디렉토리를 모두 찾아 디렉토리 변경
      - /var/lib/etcd --> /var/lib/etcd-restore
    - 옵션 추가
      - --initial-cluster-token=this-is-token
    
  - Etcd가 부팅되고 1분 정도 기다린 후 kubectl 동작 확인
    ```
    $ sudo docker ps -a | grep etcd
    # etcd가 올라오는 것을 확인해야 한다.
    ```

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
    
#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터 인프런 강의
