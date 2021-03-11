---
layout: post
title: Docker Image와 Container
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
    - [저장소 이름]/[이미지 이름]:[태그]의 형태로 구성된다.
- 도커 컨테이너
    - 이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어있는 파일시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성되고, 이 것이 바로 도커 컨테이너
    - 도커 컨테이너는 생성될 때 사용된 도커 이미지의 종류에 따라 알맞은 설정과 파일을 가지고 있기 때문에 도커 이미지의 목적에 맞도록 사용된다.
    - 도커 컨테이너는 이미지를 읽기 전용으로 사용해 때문에 이미지에서 변경된 사항만 컨테이너 계층에 저장하므로 컨테이너에서 무엇을 하든지 원래 이미지는 영향을 받지 않는다.

#### Docker Image의 레이어

- 레이어의 개념

  - 이미지 하나를 다운받더라도 여러 레이어가 다운받아지게 된다.
  - docker hub에 저장할 때 여러 레이어로 나누어 저장하기 때문이다.
  
  ![docker lifecycle](../images/docker/layer.png)
  - 왼쪽: 이미지 A를 사용하고 있을 때 이미지 B를 다운받으면 이미 레이어 A,B,C가 있기 때문에 레이어 D만 받는다.
  - 왼쪽: 이미지 A를 지운다 하더라도 이미지 B에서 A,B,C 레이어를 사용하고 있기 때문에 지워지지 않는다.
  - 오른쪽: 이미 존재하는 레이어 A,B는 새로 다운로드 받을 필요가 없다.
  
  - 도커 이미지 정보 확인
    ```
    $ docker inspect nginx // 이미지 정보 확인
    ```
    - id: docker image에 대한 id, sha256 알고리즘이 사용
    - ContainerInfo -> 포트번호, 환경 변수, CMD(컨테이너 실행할 때 사용할 명령어) 등이 있다.
    - RootFS -> FileSystem에 대한 정의
    
  - 도커 이미지 저장소 위치 확인
    ```
    $ docker info
    ```
    - 결과
      ```
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
        ```
        $ cd /var/lib/docker
        ```
      - 파일 확인
        
        ```
        builder   containers  network   plugins   swarm  trust 
        buildkit  image       overlay2  runtimes  tmp    volumes
        ```
      - image, container, overlay2 등을 확인할 수 있다.
      
      - image
        - 이미지를 관리할 수 있는 Hash 값(sha256)이 저장되어 있다.
        - iamge/overlay2/imagedb에는 layerdb에 대한 정보를 가지고 있고,
        - image/overlay2/layerdb에는 overlay2에 대한 정보를 가지고 있다.
      
      - overlay2
        ```
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
    ```
    $ docker history nginx
    ```
    - 결과 
      ```
      IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
      35c43ace9216        3 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
      <missing>           3 weeks ago         /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B                  
      <missing>           3 weeks ago         /bin/sh -c #(nop)  EXPOSE 80   
      ```
      
  - 도커 용량 확인하기        
    ```
    $ du -sh /var/lib/docker/ # 도커가 설치된 환경 용량 확인
    2.0GB /var/lib/docker/
    
    $ du -sh /var/lib/docker/image/ # 도커가 이미지에 대한 정보 저장 디렉토리
    2.7M /var/lib/docker/image/
    
    $ du -sh /var/lib/docker/overlay2 # 도커 이미지의 파일 시스템이 사용되는 실제 디렉토리
    2.0G /var/lib/docker/overlay2
    
    $ du -sh /var/lib/docker/containers # 도커 컨테이너 정보 저장 디렉토리
    136K /var/lib/docker/containers
    ```

#### 컨테이너 활용을 위한 명령어

- 포트 포워딩으로 톰캣 실행하기
  ```
  $ docker run -d --name tc -p 80:8080 tomcat
  firefox 127.0.0.1:80
  ```

- 컨테이너 내부 셸 실행
  ```
  $ docker exec -it tc /bin/bash
  
  ```

- 컨테이너 로그 확인
  ```
  $ docker logs tc #stdout, stderr
  ```

- 호스트 및 컨테이너 간 파일 복사
  ```
  $ docker cp <path> <to container>:<path>
  $ docker cp <from container>:<path> <path>
  $ docker cp <from container>:<path> <to container>:<path>
  ```

- 도커 컨테이너 모두 삭제
  ```
  $ docker stop `sudo docker ps -a -q`
  $ docker rm `sudo docker ps -a -q`
  ```

- 임시 컨테이너 생성
  ```
  $ docker run -d -p 80:8080 --rm --name tc tomcat 
  ```

- 출처: 데브옵스를 위한 쿠버네틱스 강의
