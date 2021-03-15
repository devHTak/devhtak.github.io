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

#### 도커 컨테이너 실행 연습문제

- 기존에 설치된 모든 컨테이너와 이미지 정지 및 삭제
  ```Bash
  $ docker stop `docker ps -a -q`
  $ docker rm `docker ps -a -q`
  $ docker rmi `docker images -q`
  
  # 확인
  $ docker ps -a
  $ docker images
  ```
  
- 도커 기능을 사용해 Jenkins 검색
  ```Bash
  $ docker search jenkins
  ```

- Jenkins를 사용하여 설치
  ```Bash
  $ docker pull jenkins/jenkins
  $ docker images # 설치 확인
  ```

- Jenkins 포트로 접속하여 웹서비스 열기
  ```Bash
  $ docker inspect jenkins/jenkins # 이미지 정보 확인
  $ docker create --name jk -p 8080:8080 jenkins/jenkins
  $ docker ps -a # 컨테이너 생성 확인
  $ docker start jk
  # firefox 127.0.0.1:8080 접속 확인
  ```
  - docker inspect jenkins/jenkins 결과
    ```
    "Config": {
        "Hostname": "",
        "Domainname": "",
        "User": "jenkins",
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "ExposedPorts": {
            "50000/tcp": {},
            "8080/tcp": {}
    },
    ```      

- Jenkins의 초기 패스워드 찾아서 로그인하기
  ```
  $ docker exec -it jk cat /var/jenkins_home/secrets/initialAdminPassword
  $ docker logs jk
  ```

- 출처: 용찬호 님 저자의 시작하세요! 도커/쿠버네티스
- 출처: 데브옵스를 위한 쿠버네틱스 강의
