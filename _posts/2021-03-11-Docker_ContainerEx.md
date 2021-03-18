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

#### 간단한 Spring boot 프로젝트 Docker로 실행하기

- Spring Boot Project 생성
  
  - 간단하게 spring-boot-starter-web 의존성을 추가하였다.
  - Controller 생성하였다.
    ```java
    @RestController
    public class BasicController {
        @GetMapping("/docker")
        public String docker() {
            return "docker";
        }

        @GetMapping("/hello")
        public String hello() {
            return "hello docker";
        }
    }
    ```
  - Maven 빌드하기
    - Update Project
    - maven package
      - 프로젝트 우클릭 -> run as -> maven build -> goals: package, profiles: pom.xml 설정
    - target 폴더 아래 생성된 jar파일 ghkrdls
  - Spring Boot 앱 실행
    ```
    $ java -jar target/DockerTest-0.0.1-SNAPSHOT.jar
    ```

- 도커 파일 만들기
  - dockerfile 생성
    ```
    FROM openjdk:8-jdk-alpine
    ARG JAR_FILE=./*.jar
    COPY ${JAR_FILE} app.jar
    ENTRYPOINT ["java", "-jar", "./app.jar"]
    ```
    - FROM
      - Docker에게 주어진 이미지를(태그 포함) 빌드시 기반으로 사용하도록 지시한다.
      - openjdk 중 tag가 8-jdk-alpine인 jdk를 기반으로 하여 docker 이미지를 만든다.
    - ARG
      - 빌드시 사용할 환경 변수를 선언한다.
      - Spring JAR 파일이 생성되는 위치를 변수로 선언
    - COPY
      - jar 파일을 app.jar 이름으로 복사
      - 실행할 jar 파일명을 통일하기 위해서이다.
      - Container화 할 때 Jar 파일명이 매번 달라지면 실행하기 어렵기 때문이다.
    - ENTRYPOINT
      - 이미지를 Container로 띄울 때 Jar 파일이 실행되어 Spring 서버가 구동되도록 Command를 설정
      - shell 스크립트를 직접 작성하고 ENTRYPOINT에 shell을 선언하는 것도 가능하다.
      
- dockerfile과 jar 파일 위치
  ```
  $ ls
  app.jar  dockerfile
  ```
  - 편의를 위해 jar파일 명을 app으로 수정했다.
  - jar파일과 dockerfile의 위치가 같기 때문에 JAR_FILE에 위치를 현재 폴더로 하였고, COPY에서도 같은 jar파일명으로 하였다.
- dockerfile 빌드
  ```
  $ docker build -t demo/spring-docker .
  ```
  - docker build -t \[생성할 이미지명 <group>/<artifactId>] [Dockerfile 위치]
  - 도커 이미지가 생성된 것을 확인할 수 있다.
  
  ```
  $ docker images
  REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
  demo/spring-docker   latest              8a1f21c4467c        2 minutes ago       122MB
  openjdk              8-jdk-alpine        a3562aa0b991        22 months ago       105MB
  ```
  
- 이미지 실행하기
  ```
  $ docker run -p 8080:8080 demo/spring-docker
  ```
  - Container가 구동되어 실행하는 것을 ps 명령으로 확인할 수 있다.
  ```
  $ docker ps
  CONTAINER ID        IMAGE                COMMAND                 CREATED              STATUS                        PORTS               NAMES
  24dfdd1217ab        demo/spring-docker   "java -jar ./app.jar"   About a minute ago   Exited (130) 19 seconds ago                       charming_davinci
  ```

#### 도커 이미지 푸시

- 도커 이미지 태그 변경 후 푸시
  - 먼저 dockerhub 가입을 하자
  - login 후에 id를 입력하여 권한을 주어야 한다.
  
  ```
  $ docker login
  $ docker tag spring-docker docker_id/spring-docker:v1.0
  $ docker images
  $ docker push docker_id/spring-docker:v1.0
  ```
  
  - https://hub.docker.com/에 접속하여 레파지토리에 도커가 등록됐는지 확인해본다.
  - 다운로드하여 실행할 수 있다.
    ```
    $ docker run -t -p 8080:8080 --name sd --rm docker_id/spring-docker:v1.0
    ```

- 도커 이미지 히스토리 확인
  - 도커 이미지가 어떤 히스토리를 가졌는지 확인할 수 있다.
    ```
    $ docker history docker_id/spring-docker
    ```

** 참고: 인프런 강의 중 데브옵스를 위한 도커/쿠버네티스 강연
