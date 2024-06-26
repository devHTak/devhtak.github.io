---
layout: post
title: RxJava, SSE 통신
summary: Reactive Programming
author: devhtak
date: '2021-05-15 21:41:00 +0900'
category: Reactive
---

#### 실습 프로젝트 구조

- 서버 측 프로젝트 구조
- 클라이언트 측 프로젝트 구조

#### 실습 프로젝트 개요 및 요구사항

- 개요
  - 클라이언트 측에서 서버 측에서 요청을 하면 서버 측에서 온도/습도 센서의 데이터를 실시간으로 클라이언트 측에 전송한다.
  - 클라이언트는 서버 측으로부터 전송받은 온/습도 데이터를 실시간으로 웹 브라우저에 표시한다.
  
- 요구사항
  - 서버 측에서 요청한 온/습도 데이터는 온/습도가 변할때마다 지속적으로 클라이언트에 전송 되어야 한다.
  - 온/습도 센서로부터 데이터를 가져오는 부분은 서버 측에서 랜덤으로 데이터를 생성하도록 구현한다.
  - 클라이언트 측에서 [온/습도 측정] 버튼을 누르면 온/습도 데이터가 화면에 지속적으로 표시가 되어야 한다.
  - 클라이언트 측에서 [온/습도 측정 중지] 버튼을 누르면 온/습도 데이터의 표시가 중지되어야 한다.
  - 클라이언트 측에서 [온/습도 측정] 버튼을 누르면 기존에 표시된 온/습도 데이터를 모두 지우고 새롭게 표시할 수 있어야 한다.
  
#### 실습 프로젝트의 동작 흐름

- SSE (Server Sent Event) 통신이란?
  - 사전적 의미 그대로 새 이벤트가 발생할 때 마다 서버에서 클라이언트로 데이터를 보내는 HTML 표준 기술
  - 웹 소켓과 같은 양방향 프로토콜이 아니라 서버에서 클라이언트 측으로 데이터를 보내는 단방향 프로토콜
  - 클라이언트 측에서 서버 측을 폴링할 필요가 없으므로 HTTP 오버 헤드가 적다
  - 서버에서 클라이언트로 단방향 데이터 전송에는 웹소켓보다 SSE가 더 적합하다.
  
- 동작흐름
  - 클라이언트 측에서 \[온/습도 측정] 버튼을 클릭한다.
  - 클라이언트는 서버에 SSE 통신 연결 요청을 보내고, 서버 측은 SSE 통신 연결을 맺는다.
  - 요청을 받은 RestController는 Sensor에게 데이터 구독 요청을 보내고 통지를 받는다.
  - 온/습도 데이터를 받은 RestController는 SSE Emitter에게 데이터를 전달한다.
  - SSE Emitter는 클라이언트 측에 데이터를 전송하고 클라이언트 측에 온/습도 데이터를 표시한다.

- 참고: IoT 프로젝트가 아니므로 Sensor 부분은 임의의 데이터를 전송하도록 했다.

#### 출처

케빈의 알기 쉬운 RxJava 2부

