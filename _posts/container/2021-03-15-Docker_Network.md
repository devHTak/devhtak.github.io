---
layout: post
title: Docker Network
summary: Docker
author: devhtak
date: '2021-03-15 21:41:00 +0900'
category: Container
---

#### 도커 네트워크 구조

- 도커 컨테이너 내부에서 ifconfig를 입력하면 컨테이너의 네트워크 인터페이스에 eht0과 lo 네트워크 인터페이스가 있는 것을 확인할 수 있다.
- 도커는 컨테이너에 내부 IP를 순차적으로 할당하며, 이 IP는 컨테이너를 재시작할 때마다 변경될 수 있다.
- 내부 IP는 도커가 설치된 호스트, 즉 내부 망에서만 사용할 수 있는 IP이기 때문에 외부와 연결될 필요가 있다.
  - 이 과정은 veth(virtual ethernet)이라는 네트워크 인터페이스를 생성함으로써 이뤄집니다.
  - 도커는 각 컨테이너에 외부와의 네트워크를 제공하기 위해 컨테이너마다 가상 네트워크 인터페이스를 호스트에 생성하며 이 인터페이스의 이름은 veth로 시작한다.
  - veth 인터페이스는 사용자가 직접 생성할 필요가 없으며, 컨테이너가 생성될 때 도커 엔진이 자동으로 생성한다.
  - 도커가 설치된 호스트에서 ifconfig, if addr 명령으로 확인할 수 있다.
- veth 인터페이스 뿐 아니라 docker0라는 브릿지도 존재하여 각 veth 인터페이스와 바인딩 돼 호스트의 eth0 인터페이스와 이어주는 역할을 한다.
- 
![docker lifecycle](../../images/docker/network.png)

(출처: https://joont92.github.io/docker/network-%EA%B5%AC%EC%A1%B0/)

#### 도커 네트워크 기능

- 컨테이너를 생성하면 기본적으로 docker0 브리지 외에 여러 네트워크 드라이버를 사용할 수 있다.
  - 도커 제공: bridge, host, none, container, overlay 등이 있다.
  - third party 플러그인 솔루션: weave, flannel, openvswitch 등이 있다.
  
- 도커에서 기본적으로 사용할 수 있는 네트워크 종류 알아보기
  ```
  $ docker network ls
  ```
- 네트워크 자세한 정보 살펴보기
  ```
  $ docker network inspect
  $ docker inspect --type network
  ```
  - Config 항목의 서브넷과 게이트웨이의 설정을 알 수 있다. 
  - 브릿지 네트워크를 사용중인 컨테이너의 목록을 Containers 항목에서 확인할 수 있다.
  
- 브릿지 네트워크
  - 브릿지 네트워크 생성하기
    ```
    $ docker network create --driver bridge mybridge
    ```
  - docker run or docker create 시 네트워크 설정
    ```
    $ docker run -i -t --name mynetwork_container \
    --net mybridge \
    ubuntu:14.04
    ```
  - 브릿지 네트워크와 컨테이너에 유동적으로 붙이고 뗄 수 있다.
    ```
    $ docker network disconnect mybridge mynetwork_container
    $ docker network connect mybridge mynetwork_container
    ```
    - 브릿지 네트워크와 오버레이 네트워크 같이 특정 IP 대역을 갖는 네트워크 모드에만 해당 명령어를 사용할 수 있다.
    - none 네트워크, host 네트워크에서는 사용할 수 없다.
  - 네트워크의 서브넷, 게이트웨이, IP 할당 범위 등을 임의로 설정하기 위해서는 --subnet, --ip-range, --gateway 옵션을 추가할 수 있다.
    ```
    $ docker network create --driver=bridge \
    --subnet=172.72.0.0/16 \
    --ip-range=172.72.0.0/24 \
    --gateway=172.72.0.0.1 \
    my_custom_network
    ```
    - subnet과 ip-range, gateway는 같은 대역이어야 한다.
    
- 호스트 네트워크
  - 네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 사용할 수 있다.
  - 호스트 네트워크 사용하기
    ```
    $ docker run -i -t --name mynetwork_host \
    --net host \
    ubuntu:14.0.4
    ```
    - 호스트 이름은 도커 엔진이 설치된 호스트 머신의 호스트 이름으로 설정된다.
    - 컨테이너의 네트워크를 호스트 모드로 설정하면 컨테이너 내부의 애플리케이션을 별도의 포트 포워딩 없이 바로 서비스할 수 있다.
    
- 논 네트워크
  - none은 말 그대로 아무런 네트워크를 쓰지 않는 것을 뜻한다.
  - 다음과 같이 사용하면 컨테이너는 외부 연결이 단절된다.
  ```
  $ docker run -i -t --name network_none \
  --net none \
  ubuntu:14.04
  ```
  
- 컨테이너 네트워크
  - --net 옵션에 container를 입력하면 다른 컨테이너의 네트워크 네임스페이스 환경을 공유할 수 있다.
  - 공유되는 속성은 내부 IP, 네트워크 인터페이스의 MAC 주소 등이다.
  - --net 옵션의 값으로 container:[컨테이너 ID]와 같이 입력한다.
  ```
  $ docker run -i -t -d --name network_container1 ubuntu:14.04
  $ docker run -i -t -d --name network_container2 \
  --net container:network_container1 \
  ubuntu:14.0.4
  ```
  - network_container1의 네트워크와 관련된 사항은 전부 network_container2와 같게 설정된다.
    ```
    $ docker exec network_container1 ifconfig
    $ docker exec network_container2 ifconfig
    ```
    - 해당 명령어로 확인할 수 있다.

#### 브리지 네트워크와 --net-alias

- run 명령어의 --net-alias 옵션을 함께 쓰면 특정 호스트의 이름으로 컨테이너 여러 개에 접근할 수 있다.
- 예제
  - 컨테이너 3개 설정
    ```    
    $ docker run -it -d --name network_alias_container1 \
    --net mybridge \
    --net-alias alicek106 ubuntu:16.04
    $ docker run -it -d --name network_alias_container2 \
    --net mybridge \
    --net-alias alicek106 ubuntu:16.04
    $ docker run -it -d --name network_alias_container3 \
    --net mybridge \
    --net-alias alicek106 ubuntu:16.04
    ```

  - alicek106 으로 ping 을 날려보자
    ```
    $ ping -c 1 alicek106
    ```
    - 컨테이너 3개의 IP로 각각 ping이 전송된다.
    - 매번 달라지는 IP를 결정하는 것은 별도의 알고리즘이 아닌 라운드 로빈 방식이다.
    - 도커 엔진이 내장된 DNS가 alicek106이라는 호스트 이름을 --net-alias 옵션으로 alicek106을 설정한 컨테이너로 변환해준다.

#### MacVLAN 네트워크

- MacVLAN은 호스트의 네트워크 인터페이스 카드를 가상화해 물리 네트워크 환경을 컨테이너에게 동일하게 제공
- 즉, 컨테이너는 물리 네트워크 상에서 가상의 맥 주소를 가지며, 해당 네트워크에 연결된 다른 장치와의 통신이 가능해진다.
- 공유기, 라우터, 스위치와 같은 네트워크 장비에 여러 대의 서버가 연결되어도 통신이 가능하다.
- 예제
  - 아래와 같은 네트워크 정보에서 MacVLAN 생성
    ```
    공유기의 네트워크 정보: 192.168.0.0/24
    서버 1(node01): 192.168.0.50
    서버 2(node02): 192.168.0.51
    ``` 
  
  - 두 서버에서 MacVLAN 네트워크 생성
    ```
    $ dockr network create -d mackvlan --subnet=192.168.0.0/24 \
    --ip-range=192.168.0.68/28 --gateway=192.168.0.1 \
    -o macvlan_mode=bridge -o parent=eth0 my_macvlan

    $ dockr network create -d mackvlan --subnet=192.168.0.0/24 \
    --ip-range=192.168.0.128/28 --gateway=192.168.0.1 \
    -o macvlan_mode=bridge -o parent=eth0 my_macvlan
    ```
      - -d: driver로 mackvlan 사용
      - --subnet: 컨테이너가 사용할 네트워크 정보를 입력. 네트워크 장비의 IP 대역 기본 설정을 그대로 따른다.
      - --ip-range: MacVLAN을 생성하는 호스트에서 사용할 컨테이너 IP 범위를 입력, 반드시 겹치지 않게 설정해야 한다.
      - --gateway: 네트워크에 설정된 게이트웨이를 입력
      - -o: 네트워크의 추가적인 옵션을 설정, 위에서는 모드를 브릿지 모드로 설정하였다

  - node01 에 c1, node02 에 c2 컨테이너 생성
    ```
    $ docker run -it --name c1 --hostname c1 \
    --network my_macvlan ubuntu:14.04
    ```
    ```
    $ docker run -it --name c2 --hostname c2 \
    --network my_macvlan ubuntu:14.04
    ```
    
  - c1, c2 끼리 통신이 가능해진다.

- 출처: https://joont92.github.io/docker/network-%EA%B5%AC%EC%A1%B0/
- 출처: 시작하세요. 도커/쿠버네티스 
