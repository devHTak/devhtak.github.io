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
    - 빌드 컨텍스트 하위 폴더도 포함되기 때문에 불필요한 파일이 있으면 성능이 떨어진다.
  - .dockerignore 파일을 작성하면 빌드 시 명시된 이름의 파일을 컨텍스트에서 제외한다.
    ```
    $ vi .dockerignore
    test2.html
    *.html
    */*.html
    !test.html?
    ```
    - *: 모든 파일을 뜻한다.
    - ?: 임의의 1자리 문자가 들어가는 파일
    - !: 해당 파일은 제외에서 제외한다.

- Dockerfile을 이용한 컨테이너 생성과 커밋
  - Dockerfile에서 명령어 한 줄이 실행될 때마다 이전 Step에서 생성된 이미지에 의해 새로운 컨테이너가 생성되며, Dockerfile에 적힌 명령어를 수행하고 다시 새로운 이미지 레이어로 저장된다.
  - 따라서, 이미지가 생성되면 Dockerfile에 명령어 줄 수만큼 레이어가 존재하게 되며, 중간에 컨테이너도 같은 수만큼 생성되고 삭제된다.

- 캐시를 이용한 이미지 빌드
  - 한번 이미지 빌드를 마치고 난 뒤 다시 같은 빌드를 진행하면 이전의 이미지 빌드에서 사용했던 캐시를 사용한다.
    ```
    $ cp Dockerfile Dockerfile2
    $ docker build -f Dockerfile2 -t mycache:0.0 ./
      # step 별로 Using Cache 내용을 확인할 수 있다.
    ```
  - 하지만 캐시가 불필요한 경우가 있다.
    - git에서 clone해 오는 경우, 캐시에 남아있는 소스를 그대로 사용하면 안된다.
    - --no-cache옵션을 추가하면 된다.
      ```
      $ docker build --no-cache -t mybuild:0.1
      ```
  - 캐시를 사용할 이미지를 직접 지정할 수도 있다.
    ```
    $ docker build --cache-from nginx -t my_extend_nginx:0.0
    ```

- 멀티 스테이지를 이용한 Dockerfile 빌드하기
  - 개발한 어플리케이션을 사용하기 위해서는 해당 언어에 맞는 의존성 패키지와 라이브러리가 필요하다.
  - 17.0.5 버전 이상을  사용하는 도커 엔진이라면, 이미지의 크기를 줄이기 위한 멀티 스테이지 빌드 방법을 사용할 수 있다.
  - 멀티 스테이지 빌드는 하나의 Dockerfile 안에 여러개의 FROM 이미지를 정의함으로써 빌드 완료 시 최종적으로 생성될 이미지의 크기를 줄이는 역할을 한다.
  - 멀티 스테이지 빌드는 반드시 필요한 실행 파일만 최종 이미지 결과물에 포함시킴으로써 이미지를 크게 줄일 때 유용하게 사용할 수 있다.
    ```
    FROM golang
    ADD main.go /root
    WORKDIR /root
    RUN go build -o /root/mainApp /root/main.go
    
    FROM alpine:latest
    WORKDIR /root
    COPY --from=0 /root/mainApp .
    CMD ["./mainApp"]
    ```
    - 2개의 FROM을 통해 2개의 이미지가 명시되었다.
    - 첫 번째 FROM을 통해 명시된 golang 이미지는 이전과 동일하게 main.go 파일을 /root/mainApp으로 빌드하였다.
    - 두 번째 FROM 아래에서 사용된 COPY 명령어는 첫 번째 FROM에서 사용된 이미지의 최종 상태에 존재하는 /root/mainApp 파일을 두 번째 이미지인 alpine에 복사한다.
      - alpine 이미지는 매우 작지만 프로그램 실행에 필요한 런타임 요소가 포함되어 있는 리눅스 배포판 이미지이다.
    - 이 때, --from=0은 첫 번째 FROM에서 빌드된 이미지의 최종 상태를 의미한다.
    - 즉, 첫 번째 FROM 이미지에서 빌드한 /root/mainApp 파일을 두번째의 FROM 절에 명시된 이미지를 alpine:latest 이미지에 복사하는 것
    ```
    $ docker build . -t go_helloworld:multi-stage
    $ docker images    
    ```
    - 이미지의 크기가 줄어든 것을 확인할 수 있다.

#### 기타 Dockerfile 명령어

- ENV
  - Dockerfile에 사용될 환경변수를 지정한다.
  - ${ENV_NAME}, $ENV_NAME 형태로 사용할 수 있다.
  ```
  ENV test /home
  ```
    - test라는 변수에 /home이라는 값을 설정하였다.
  - run 명령어에서 -e 옵션을 사용해 같은 이름의 환경변수를 사용하면 기존 값은 덮어진다.

- VOLUME
  - 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉터리를 설정한다.
  - JSON 형식으로 여러개를 사용하거나 나열하여 사용할 수 있다.
    - VOLUME ["/home/dir", "/home/dir2"]
    - VOLUME /home/dir /home/dir2
  - 컨테이너를 생성하고 볼륨의 목록을 확인해 보면 볼륨이 생성된 것을 알 수 있다.

- ARG
  - build 명령어를 실행할 때 추가로 입력을 받아 Dockerfile 내에서 사용될 변수의 값을 설정한다.
  - 기본값을 지정할 수도 있다.
    ```
    FROM ubuntu:14.04
    ARG my_arg
    ARG my_arg2=value2
    RUN touch ${my_arg2}/mytouch
    ```
  - 빌드할 때 --build-arg 옵션을 사용하여 key=value 형태로 입력할 수 있다.
    ```
    $ docker build --build-arg my_arg=/home -t my_arg:0.0 .
    ```
  - ARG, ENV의 값을 사용하는 방법은 ${}으로 같으므로 Dockerfile에서 ARG로 설정한 변수를 ENV에서 같은 이름으로 다시 정의하면 --build-arg 옵션에서 설정하는 값은 ENV에 의해 덮어진다.
    
- USER
  - USER 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자 권한으로 실행된다.
  - 기본적으로 root 권한으로 실행되는 데 필요 없는 경우 사용할 수 있다.
    ```
    //...
    RUN groupadd -r author && useradd -r -g author devhtak
    USER devhtak
    //...
    ```
  
- OnBuild
  - 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile로 생성될 때 실행할 명령어를 추가한다.
    ```
    FROM ubuntu:14.04
    RUN echo "this is onbuild test!"
    ONBUILD RUN echo "onbuild!" >> /oubuild_file
    ```
    - 처음으로 빌드를 할 때에는 해당 명령어로 이뤄진 레이어는 생기지만 onbuild_file이 생성되지는 않는다.
      ```
      $ docker build -t onbulid_test:0.0 .
      ```
    - 해당 이미지를 기반으로 빌드를 하면 onbuild_file이 생성되는 것을 확인할 수 있다.
      ```
      FROM onbuild_test:0.0
      RUN echo "this is child image!"
      ```
      ```
      $ docker build -t onbuild_test:0.1 .
      $ docker run -it --rm onbuild_test0.1 ls /onbuild_file
      onbuild_file
      ```
      
- Stopsignal
  - 컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정한다.
  - 아무것도 설정하지 않으면 기본적으로 SIGTERM으로 설정되지망 Dockerfile에 STOPSIGNAL을 정의해 컨테이너가 종료되는 사용될 신호를 선택할 수 있다.
  - Dockerfile의 STOPSIGNAL은 docker run 명령어에서 --stop-signal 옵션으로 개별적으로 설정할 수 있다.

- HealthCheck
  - 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정
  - 컨테이너 내부에서 동작중인 애플리케이션의 프로세스가 종료되지는 않았으나 애플리케이션이 동작하고 있지 않은 상태를 방지하기 위해 사용
  ```
  FROM nginx
  RUN apt-get update -y && apt-get install curl -y
  HEALTHCHECK --interval=1m --timeout=3s --retrieve=3 CMD curl -f http://localhost || exit 1
  ```
    - --interval로 설정한 시기마다, CMD curl 부분이 상태를 체크하는 명령어가 된다.
    - --timeout으로 설정한 시간을 초과하면 실패한 것으로 간주하고 실패한 경우 --retries의 횟수만큼 명령어를 반복한다.
    - --retries에 설정된 횟수만큼 상태 체크에 실패하면 해당 컨테이너는 unhealthy 상태로 설정

- Shell
  - Dockerfile에서 기본적으로 사용하는 쉘은 리눅스에서 "/bin/sh -c", 윈도우에서 "cmd /S /C"이다.
  - 사용하고자 하는 다른 쉘을 사용하고자 할 때, SHELL을 사용한다.

- ADD 와 COPY
  - ADD는 외부 URL 및 tar 파일에서도 파일을 추가할 수 있다.
    ```
    ADD https://raw.githubsercontent.com/alicek106/mydockerrepo/matser/test.htmlk /home/
    ADD test.tar /home
    ```
  - COPY는 로컬 디렉터리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할을 한다.
    ```
    COPY test.html /home/
    COPY ["test.html", "/home/"]
    ```
  - ADD 보다는 COPY를 선호한다. ADD를 했을 경우 정확히 어떤 파일인지 확인하기 어렵기 때문이다.

- ENTRYPOINT와 CMD
  - 같은 점
    - ENTRYPOINT는 CMD와 동일하게 컨테이너가 시작될 때 수행할 명령을 지정한다.
  - 차이점
    - ENTRYPOINT는 커맨드를 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있다는 점에서 다르다.
  - ENTRYPOINT를 활용한 스크립트 실행
    ```
    $ docker run -it --name entrypoint_sh --entrypoint='/test.sh' ubuntu:14.04 /bin/bash
    ```
    - 실행할 스크립트는 컨테이너 내부에 존재해야 한다.
    - 없는 경우 COPY, ADD를 사용하여 복사한다.
    ```
    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install apache2 -y
    ADD entrypoint.sh /entrypoint.sh
    RUN chmod +x /entrypoint.sh
    ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
    ```

  - 이미지 빌드 시 이미지 작동 단계
    - 어떤 설정 및 실행이 필요한지에 대해 스크립트로 정리
    - ADD 또는 COPY로 스크립트를 이미지로 복사
    - ENTRYPOINT를 이 스크립트로 설정
    - 이미지를 빌드해 사용
    - 스크립트에서 필요한 인자는 docker run 명령어에서 cmd로 entrypoint의 스크립트에 전달
  
  - JSON 배열 형태와 일반 형식의 차이점
    - ENTRYPOINT, CMD 에서 사용할 명령어를 /bin/sh로 사용할 수 없으면, JSON 벼열의 형태로 사용해야 한다.

#### Dockerfile로 빌드할 때 주의점

- 좋은 습관
  - 하나의 명령어를 \(역슬래시)로 나누어 가독성을 높일 수 있도록 작성하자.
  - 사용하지 않는 파일은 .dockerignore 파일을 작성해 불필요한 파일을 빌드 컨텍스트에 포함하지 않도록 하자.
  - 빌드 캐시를 이용해 기존에 사용했던 이미지 레이어를 재사용하자.

- 주의점
  ```
  FROM ubuntu:14.04
  RUN mkdir /test
  RUN fallocate -l 100m /test/dummy
  RUN rm /test/dummy
  ```
  - 레이어(명령어)를 살펴보면 100m 크기의 /test/dummy를 만들었다가 삭제한다.
  - 파일을 삭제했다 하더라도 레이어로 만든 기록이 있기 때문에 여전히 이미지의 크기가 높다.
  - && 으로 실행할 내용을 하나로 묶어서 사용하면 된다.
  ```
  FROM ubuntu:14.04
  RUN mkdir /test && \
  fallocate -l 100m /test/dummy && \
  rm /test/dummy
  ```
  - RUN이 하나의 이미지 레이어가 된다는 것을 생각해보면 간단한 해결책이다. 
  - 묶어서 사용하는 명령어는 하나이 레이어로 되며, 레이어 수를 줄일 수 있다.


** 참고: 용찬호 님의 시작하세요! 도커/쿠버네티스
