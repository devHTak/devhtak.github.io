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
    - eth0 (10.0.2.4) -> docker0 (127.17.0.1 - bridge) -> veth0(172.17.0.2 - container1), veth1(172.17.0.3 - container2)

  - 컨테이너 간의 인터페이스를 공유하는 구조
    - eth0 (10.0.2.4) -> docker0 (172.17.0.1 - bridge) -> veth0 (172.17.0.2 - pause, container1, container2)
  
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
  |RouteDistribution|BGP 프로토콜 사용, 네트워크 세그먼트에 분할된 클러스터를 구축하려는 경우 사용, 인터넷에서 라우팅 및 연결 가능성 정보를 교환하도록 설계된




** 데브옵스(DevOps)를 위한 쿠버네티스 마스터
