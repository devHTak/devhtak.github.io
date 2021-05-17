---
layout: post
title: Docker Image
summary: Docker
author: devhtak
date: '2021-03-11 21:41:00 +0900'
category: Container
---

#### 도커 이미지와 컨테이너

- 도커 엔진에서 사용하는 기본단위는 이미지와 컨테이너이며, 핵심

- 도커 이미지
    - 컨테이너를 생성할 때 필요한 요소로 가상머신에서의 iso와 비슷한 개념
    - 여러 개의 계층으로 된 바이너리 파일로 존재하고, 컨테이너를 생성하고 실행할 때 읽기 전용으로 사용된다.
    - \[저장소 이름]/\[이미지 이름]:\[태그]의 형태로 구성된다.
    
#### Docker Image의 레이어

- 레이어의 개념

  - 이미지 하나를 다운받더라도 여러 레이어가 다운받아지게 된다.
  - docker hub에 저장할 때 여러 레이어로 나누어 저장하기 때문이다.
  
  ![docker lifecycle](../images/docker/layer.png)
  - 왼쪽: 이미지 A를 사용하고 있을 때 이미지 B를 다운받으면 이미 레이어 A,B,C가 있기 때문에 레이어 D만 받는다.
  - 왼쪽: 이미지 A를 지운다 하더라도 이미지 B에서 A,B,C 레이어를 사용하고 있기 때문에 지워지지 않는다.
  - 오른쪽: 이미 존재하는 레이어 A,B는 새로 다운로드 받을 필요가 없다.
  
  - 도커 이미지 정보 확인
    ```Bash
    $ docker inspect nginx // 이미지 정보 확인
    ```
    - id: docker image에 대한 id, sha256 알고리즘이 사용
    - ContainerInfo -> 포트번호, 환경 변수, CMD(컨테이너 실행할 때 사용할 명령어) 등이 있다.
    - RootFS -> FileSystem에 대한 정의
    
  - 도커 이미지 저장소 위치 확인
    ```Bash
    $ docker info
    ```
    - 결과
      ```Bash
      Client:
       Debug Mode: false

      Server:
       Containers: 0
        Running: 0
        Paused: 0
        Stopped: 0
       Images: 1
       Server Version: 19.03.6
       Storage Driver: overlay2
        Backing Filesystem: extfs
        Supports d_type: true
        Native Overlay Diff: true
       Logging Driver: json-file
       Cgroup Driver: cgroupfs
       Plugins:
        Volume: local
        Network: bridge host ipvlan macvlan null overlay
        Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
       Swarm: inactive
       Runtimes: runc
       Default Runtime: runc
       Init Binary: docker-init
       containerd version: 
       runc version: 
       init version: 
       Security Options:
        apparmor
        seccomp
         Profile: default
       Kernel Version: 4.18.0-20-generic
       Operating System: Ubuntu 18.04.2 LTS
       OSType: linux
       Architecture: x86_64
       CPUs: 2
       Total Memory: 3.852GiB
       Name: server1-VirtualBox
       ID: 72TD:JRNP:R45N:YXZ5:Y2JZ:T7VW:OYT4:B6TH:CRUB:FRS5:Y7MN:N5GM
       Docker Root Dir: /var/lib/docker
       Debug Mode: false
       Registry: https://index.docker.io/v1/
       Labels:
       Experimental: false
       Insecure Registries:
        127.0.0.0/8
       Live Restore Enabled: false
      ```
    - 레이어 저장소 확인
      - 도커 설치 디렉토리 이동
        ```Bash
        $ cd /var/lib/docker
        ```
      - 파일 확인
        
        ```Bash
        builder   containers  network   plugins   swarm  trust 
        buildkit  image       overlay2  runtimes  tmp    volumes
        ```
      - image, container, overlay2 등을 확인할 수 있다.
      
      - image
        - 이미지를 관리할 수 있는 Hash 값(sha256)이 저장되어 있다.
        - iamge/overlay2/imagedb에는 layerdb에 대한 정보를 가지고 있고,
        - image/overlay2/layerdb에는 overlay2에 대한 정보를 가지고 있다.
      
      - overlay2
        ```Bash
        root@server1-VirtualBox:/var/lib/docker/overlay2# ls
        1b5bbf96c8c8fd658eb619f8eb044739c6ce1487d2e96e91241e82289d1f4457 # 레이어 변경 사항 저장
        356f4333d1a5ab8a97f7715b86cdaf566523d271605cfff48648d85524e12609 # 레이어 변경 사항 저장
        7537f4c622c051a15e06e949f9c0ef18bbbe1d0963254090dcfb5707ae1befe6 # 레이어 변경 사항 저장
        8c43fc93cc50d8b7cda040be1a55ee970d18d824c6016ff3890e940b7400a217 # 레이어 변경 사항 저장
        cb6e95fe88958e4c00ee313f12e281f58e5eb55e52f6ad98f7d4defe5b44da86 # 레이어 변경 사항 저장
        cdf85b75556fb17a210ef6b891a41fa02440c0347201cec6a93267cc8af28be4 # 레이어 변경 사항 저장
        l # 원본 레이어 저장
        ```
        - overlay2에 이미지에 대한 레이어가 저장되어 있다.
        - overlay2에는 변경 사항이 저장되어 있는데, 실질적으로 overlay2/l 디렉터리 하위에 저장되어 있다.
        - 실제로 container, image는 미미하고 overlay2에 정보가 저장되기 때문에 많은 용량을 차지한다.
        
  - 도커 히스토리 확인
    ```Bash
    $ docker history nginx
    ```
    - 결과 
      ```Bash
      IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
      35c43ace9216        3 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
      <missing>           3 weeks ago         /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B                  
      <missing>           3 weeks ago         /bin/sh -c #(nop)  EXPOSE 80   
      ```
      
  - 도커 용량 확인하기        
    ```Bash
    $ du -sh /var/lib/docker/ # 도커가 설치된 환경 용량 확인
    2.0GB /var/lib/docker/
    
    $ du -sh /var/lib/docker/image/ # 도커가 이미지에 대한 정보 저장 디렉토리
    2.7M /var/lib/docker/image/
    
    $ du -sh /var/lib/docker/overlay2 # 도커 이미지의 파일 시스템이 사용되는 실제 디렉토리
    2.0G /var/lib/docker/overlay2
    
    $ du -sh /var/lib/docker/containers # 도커 컨테이너 정보 저장 디렉토리
    136K /var/lib/docker/containers
    ```
#### 도커 레지스트리(hub)

- 도커 레지스르리에는 사용자가 사용할 수 있도록 데이터베이스를 통해 image를 제공하고 있다.
- 누구나 이미지를 만들어 푸시할 수 있으며 푸시된 이미지는 다른 사람들에게 공유 가능하다.

- 이미지 찾기
  - 명령어로 CLI 형태로 찾을 수 있다.
    ```
    $ sudo docker search tomcat
    ```
  - docker hub 사용하기
    - https://hub.docker.com/
    - 보통 이름/이미지이름 으로 되어 있다.
    
- hub에서 다운로드
  - 이미지 다운로드
    ```
    $ docker pull mysql
    ```
  - 이미지 다운로드 + 실행
    ```
    $ docker run -d -p 8080:8080 --name console/tomcat-7.0
    ```
  - 이미지 확인
    ```
    $ docker images
    ```
    
#### 간단한 Spring boot 프로젝트 Docker로 이미지 만들기

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
    - target 폴더 아래 생성된 jar파일 확인
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

#### 도커 허브로 이미지 푸시

- 도커 이미지 태그 변경 후 푸시
  - 먼저 dockerhub 가입을 하자
  - login 후에 id를 입력하여 권한을 주어야 한다.
  
  ```
  $ docker login
    # 로그인
  $ docker tag spring-docker docker_id/spring-docker:v1.0
    # spring-docker의 태그 변경, Image ID는 같으나 태그만 변경된다.
  $ docker images
    # 이미지 확인
  $ docker push docker_id/spring-docker:v1.0
    # docker hub로 이미지 푸시
    # push할 때 이미 있는 레이어는 Mounted from ..으로 push되지 않고, 생성된 이미지만 push된다.
  ```
  
  - https://hub.docker.com/에 접속하여 레파지토리에 도커가 등록됐는지 확인해본다.
  - 다운로드하여 실행할 수 있다.
    ```
    $ docker run -t -p 8080:8080 --name sd --rm docker_id/spring-docker:v1.0
    ```

- 도커 이미지 히스토리 확인
  - 도커 이미지가 어떤 히스토리를 가졌는지 확인할 수 있다.
  - 도커 이미지를 생성할 때 사용된 명령어들을 확인할 수 있다.
    ```
    $ docker history docker_id/spring-docker
    ```

#### private registry server 구현 및 사용

- private registry 만들기 (registry 이미지 사용)
  ```
  $ docker run -d --name docker-registry -p 5000:5000 registry
  ```

- private registry에 이미지 푸시하기
  ```
  $ sudo docker tag spring-docker 127.0.0.1:5000/spring-docker
    # 127.0.0.1을 태그로 새로 만들었다.
  $ sudo docker push 127.0.0.1:5000/spring-docker
    # 신규 생성한 registry 서버이기 때문에 모든 레이어가 push되는 것을 확인할 수 있다.
  ```
  
- private registry에서 pull하기
  ```
  $ docker pull 127.0.0.1:5000/sprint-docker
  ```
  
- 인증 관련 참고 링크: https://docs.docker.com/registry/configuration/#auth

** 출처: 용찬호님 시작하세요! 도커/쿠버네티스

** 출처: 데브옵스를 위한 쿠버네틱스 강의
