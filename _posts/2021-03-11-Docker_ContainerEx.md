---
layout: post
title: Docker Container 생성 예제
summary: Docker
author: devhtak
date: '2021-03-11 21:41:00 +0900'
category: Container
---

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
  
#### 환경변수를 사용하여 MySQL 서비스 구축하기

- 실행 할때에 -e 옵션으로 환경변수를 사용할 수 있다.
  
- 환경변수 사용하여 데이터 전달하기
  ```
  $ docker run -d --name nx -e env_name=test1234 nginx
  $ docker exec -it nx bash
  > printenv
  HOSTNAME=b5ef878dea6d
  PWD=/
  PKG_RELEASE=1~buster
  HOME=/root
  NJS_VERSION=0.5.2
  TERM=xterm
  SHLVL=1
  env_name=test1234
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  NGINX_VERSION=1.19.8
  _=/usr/bin/printenv
  > printenv env_name
  test1234
  ```
  
- MySQL 서비스 구축하기
  ```
  $ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD='test1234' -d mysql
  $ docker exec -it ms mysql -u root -p
  Enter password: 
  ```
  
#### 볼륨 마운트하여 Jupytor LAB 서비스 구축

- 명령어 형식
  ```
  $ docker run -v <호스트 경로>:<컨테이너 내 경로>:<권한> # /tmp:/home/user:ro
  ```
  - 권한의 종류
    - ro: 읽기 전용(read only)
    - rw: 읽기 및 쓰기(read write)

- nginx로 볼륨 마운트하기
  ```
  $ docker run -d -p 80:80 --rm -v /var/www:/usr/share/nginx/html:ro
  ```

- 현재 디렉토리를 사용하여 notebook 컨테이너 실행
  ```
  $ mkdir jupyternotebook
  $ cd jupyternotebook
  $ docker run --rm -p 8080:8888 -e JUPYTOR_ENABLE_LAB=yes -v "$PWD":/home/jovyan/work:rw jupyter/datascience-notebook
  ```
  - "$PWD" 를 사용하면 현재 디렉터리로 설정한다.
