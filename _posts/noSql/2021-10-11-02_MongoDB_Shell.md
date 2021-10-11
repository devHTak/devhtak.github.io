---
layout: post
title: 도커를 활용한 MongoDB 설치 및 쉘 사용
summary: JUnit 사용법
author: devhtak
date: '2021-10-11 21:41:00 +0900'
category: No SQL
---

#### Docker로 MongoDB 설치

- MongoDB 이미지 다운로드
  ```
  $ docker pull mongo
  ```
  - docker hub에서 이미지 다운르드 
  - :version 을 명시하지 않으면 최신 버전의 이미지를 받는다

- MongoDB 컨테이너 생성 및 실행
  ```
  $ docker run --name mongodb-container -v ~data:data/db --p 27017:27017 -d mongo
  ```
  - mongo 이미지 컨테이너 생성 및 백그라운드 실행(-d)
  - 27017(host):27017(container) 포트 포워딩(-p)
  - 컨테이너 이름 mongodb-container(--name)
  - 호스트(컨테이너를 구동하는 로컬 컴퓨터)의 ~/data 디렉터리와 컨테이너의 /data/db 디렉터리를 마운트(-v)

- MongoDB 컨테이너 접속
  ```
  $ docker exec -it mongodb-container /bin/bash
  ```

- MongoDB 시작, 중지, 재시작
  ```
  # MongoDB Docker 컨테이너 중지
  $ docker stop mongodb-container
  # MongoDB Docker 컨테이너 시작
  $ docker start mongodb-container
  # MongoDB Docker 컨테이너 재시작
  $ docker restart mongodb-container
  ```
  
#### 출처
- Docker 를 활용한 Mongodb 설치 및 실행: https://poiemaweb.com/docker-mongodb
- 
