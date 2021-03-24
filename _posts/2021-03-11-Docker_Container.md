---
layout: post
title: Docker Container
summary: Docker
author: devhtak
date: '2021-03-11 21:41:00 +0900'
category: Container
---

#### 도커 컨테이너

- 도커 엔진에서 사용하는 기본단위는 이미지와 컨테이너이며, 핵심

- 도커 컨테이너
    - 이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어있는 파일시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성되고, 이 것이 바로 도커 컨테이너
    - 도커 컨테이너는 생성될 때 사용된 도커 이미지의 종류에 따라 알맞은 설정과 파일을 가지고 있기 때문에 도커 이미지의 목적에 맞도록 사용된다.
    - 도커 컨테이너는 이미지를 읽기 전용으로 사용해 때문에 이미지에서 변경된 사항만 컨테이너 계층에 저장하므로 컨테이너에서 무엇을 하든지 원래 이미지는 영향을 받지 않는다.

#### 컨테이너 활용을 위한 명령어

- 포트 포워딩으로 톰캣 실행하기
  ```Bash
  $ docker run -d --name tc -p 80:8080 tomcat
  firefox 127.0.0.1:80
  ```
  - -d: 백그라운드 실행
  - --name: 컨테이너 이름 설정
  - -p: 포트 포워딩 out:in 형태

- 컨테이너 내부 셸 실행
  ```Bash
  $ docker exec -it tc /bin/bash
  
  ```
  - 컨테이너 안으로 들어가서 확인할 수 있다.
  - -it: input terminal로 terminal을 열어달라는 의미
  - 결과
    ```Bash
    root@server1-VirtualBox:~# docker exec -it nx /bin/bash
    root@eb6bf9e2a68a:/# ls
    bin   docker-entrypoint.d   home   media  proc	sbin  tmp boot  docker-entrypoint.sh  lib    mnt	  root	     srv   usr  dev   etc
    ```
    - host명이 container id로 변경되었다.

- 컨테이너 로그 확인
  ```Bash
  $ docker logs tc #stdout, stderr
  ```
  - stdout, stderr가 logs로 출력된다.

- 호스트 및 컨테이너 간 파일 복사
  ```Bash
  $ docker cp <path> <to container>:<path>
  $ docker cp <from container>:<path> <path>
  $ docker cp <from container>:<path> <to container>:<path>
  ```
  - 도커 컨테이너 내부에 경우 <container>:<path>를 사용해야 하고, local인 경우 <path>로 사용할 수 있다.
  - 예제
  
    ```Bash
    root@server1-VirtualBox:~# echo test1234 > test.txt # test.txt 생성
    root@server1-VirtualBox:~# cat test.txt # 생성된 파일 확인
    test1234
    root@server1-VirtualBox:~# docker cp test.txt nx:/ # 파일을 nx 컨테이너의 루트로 이동
    root@server1-VirtualBox:~# docker exec -it nx cat /test.txt # nx 내에 test.txt 확인
    test1234
    root@server1-VirtualBox:~# docker cp nx:/test.txt ./test2.txt # nx 컨테이너에 있는 파일을 로컬로 이동
    root@server1-VirtualBox:~# cat test2.txt # 확인
    test1234
    ```

- 도커 컨테이너 모두 삭제
  ```Bash
  $ docker stop `docker ps -a -q` # docker ps -a로 조회된 컨테이너가 모두 중지한다.
  $ docker rm `docker ps -a -q` # docker ps -a로 조회된 컨테이너를 모두 삭제한다.
  ```

- 임시 컨테이너 생성
  ```Bash
  $ docker run -d -p 80:8080 --rm --name tc tomcat 
  ```

#### 포트 포워딩(포트 매핑)

```
$ docker run -p <host port number>:<container port number>/protocol [Image Name] [Other Options]
```
- -p 옵션을 추가하여 포트포워딩을 설정할 수 있다.
  - host port number: 호스트 시스템에서 사용될 포트 번호
  - container port number: 컨테이너 내에서 사용될 포트 번호
  - protocol: 프로토콜 유형

```
# 포트포워딩 설정하여 컨테이너 생성
$ docker container run -p 8080:80 nginx

# 포트 listen 확인
$ netstat -nlp | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      26113/docker-proxy-
```

- -p 8080:80 
  - 호스트 IP 8080번 포트로 접근 -> 8080번 포트는 컨테이너의 80번 포트로 포워딩 -> 웹 서버 접근
- 포트 listen을 확인해 보면 docker-proxy 라는 프로세스가 해당 포트를 listen 하고 있음을 볼 수 있다
  - 이는 간단히 docker host 로 들어온 요청을 해당하는 컨테이너로 넘기는 역할만을 수행하는 프로세스이다
  - 컨테이너에 포트포워딩이나 expose를 설정했을 경우 같이 생성되는 프로세스이다

- 그렇지만 중요한것은, 실제로 포트포워딩을 이 docker-proxy가 담당하는 것이 아니라, host PC iptables 에서 관리한다는 점이다

```
$ iptables -t nat -L -n

Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  tcp  --  172.17.0.4           172.17.0.4           tcp dpt:80

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.17.0.4:80
```

- 보다시피 모든 요청을 DOCKER Chain 으로 넘기고, DOCKER Chain 에서는 DNAT를 통해 포트포워딩을 해주고 있음을 볼 수 있다
- 이 iptables 룰은 docker daemon이 자동으로 한다
- docker-proxy 는 iptables가 어떠한 이유로 NAT를 사용하지 못하게 될 경우 사용된다고 한다


** 출처: https://joont92.github.io/docker/network-%EA%B5%AC%EC%A1%B0/
** 출처: 용찬호 님 저자의 시작하세요! 도커/쿠버네티스
** 출처: 데브옵스를 위한 쿠버네틱스 강의
