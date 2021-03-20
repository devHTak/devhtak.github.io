---
layout: post
title: Dockerfile
summary: Docker
author: devhtak
date: '2021-03-20 21:41:00 +0900'
category: Container
---

#### Dockerfile 역할

- 도커는 완성된 이미지를 생성하기 위해 컨테이너에 설치해야 하는 패키지, 추가해야 하는 소스 코드, 실행해야 하는 명령어와 쉘 스크립트 등을 하나의 파일에 기록해두면 도커는 이 파일을 읽어 컨테이너에서 작업을 수행한 뒤 이미지로 만들어 낸다.
- 이러한 작업을 기록한 파일의 이름을 Dockerfile이라 한다.
- 빌드 명령어는 Dockerfile을 읽어 이미지를 생성한다.
- Dockerfile을 활용하면 애플리케이션의 빌드 및 배포를 자동화할 수 있다.
- Docker hub에 이미지 자체를 배포하는 대신 이미지 생성 방법을 작성한 Dockerfile을 배포할 수도 있다.

#### Dockerfile 작성

- 예제: 웹 서버 이미지 생성
  - test.html 생성
    ```
    $ echo test >> test.html
    ```
  - Dockerfile 생성
    ```
    $ vi Dockerfile
    FROM ubuntu:14.0.4
    MAINTAINER devHTak
    LABEL "purpose"="practice"
    RUN apt-get update
    RUN apt-get install apache2 -y
    ADD test.html /var/www/html
    WORKDIR /var/www/html
    RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
    EXPOSE 80
    CMD apachectl -DFOREGROUND
    ```
    - 한줄에 하나의 명령어로 이루어져 있으며, 소문자로 작성해도 되나 보통 대문자로 작성한다.
    - 명령어는 위에서 아래로 한줄씩 실행된다.
    
    - FROM
      - 생성할 이미지의 베이스가 될 이미지를 뜻한다. 
      - FROM 명령어는 Dockerfile을 작성할 때 반드시 한 번 이상 입력해야 하며, 이미지 이름의 포맷은 docker run 명령어에서 이미지 이름과 동일하다.
      - 사용하려는 이미지가 없는 경우 hub에서 pull한다.
    - MAINTAINER
      - 이미지를 생성한 개발자의 정보(이메일 주소)를 나타낸다.
      - 도커 1.13.0 버전 이후로 사용하지 않는다. 대신, LABEL을 사용한다.
        ```
        LABEL maintainer "devHTak <devHTak@gmail.com>"
        ```
    - LABEL
      - 이미지에 메타 데이터를 추가한다.
      - 메타데이터는 key value 형태로 저장되며, 여러 개의 메타데이터를 생성할 수 있다.
      - docker inspect 명령어로 이미지의 정보를 구해서 확인할 수 있다.
    - RUN
      - 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행한다.
      - 예제에서는 아파치 웹 서버가 설치된 이미지가 생성된다.
      - RUN \["실행 가능한 파일", "명령줄 인자1", "명령줄 인자2", ...]
        - 이와 같은 방식으로 JSON 배열 형태로 명령어를 입력할 수 있다.
    - ADD
      - 파일을 이미지에 추가한다.
      - 추가하는 파일은 Dockerfile이 위치한 디렉터리인 컨텍스트(Context)에서 가져온다.
      - 예제에서는 test.html 파일을 이미지 내에 /var/www/html에 추가한다.
      - ADD \["추가할 파일 이름", ..., "컨테이너에 추가될 위치"]
        - JSON 배열 형태로 파일을 추가할 수 있다.
    - WORKDIR
      - 명령어를 실행할 디렉터리를 나타낸다.
      - /bin/bash에서 cd 명령어와 같은 역할을 한다.
    - EXPOSE
      - Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정한다.
      - 이미지를 컨테이너로 만들 때 포트 포워딩을 -P(Publish)로 하면 해당 expose로 설정된 포트로 접근할 수 있다.
    - CMD
      - 컨테이너가 실행될 때마다 실행할 명령어(커맨드)를 설정한다.
      - Dockerfile에서 한번만 사용할 수 있다.
      - 예제에서는 컨테이너가 실행될 때 마다 apache가 실행되도록 했다.
      - CMD \["실행 가능한 파일", "명령줄 인자1", ...] 형태로도 사용 가능하다.
      - CMD 대신 ENTRYPOINT를 사용할 수 있다.

  - 빌드
    ```
    $ docker build -t mybuild:0.0 ./
    ```
    - -t 옵션을 이용하여 빌드된 이미지의 이름을 지정할 수 있다.
      - mybuild:0.0 라는 이름의 이미지 생성
      - -t 옵션을 사용하지 않으면 16진수로 된 임의의 이미지명이 생성되어 사용하는 것이 좋다.
    - ./ 현재 디렉터리에 Dockerfile이 있다는 것을 명시했다.

#### 빌드 과정

- 빌드 컨텍스트
  - 이미지 빌드를 시작하면, 도커는 가장 먼저 빌드 컨텍스트를 읽어들인다.
    - 빌드 컨텍스트란, Dockerfile이 위치한 디렉터리
    - 빌드 컨텍스트는 이미지를 생성하는 데 필요한 각종 파일, 소스코드, 메타데이터 등을 담고 있는 디렉터리를 의미한다.

** 참고: 용찬호 님의 시작하세요! 도커/쿠버네티스



