---
layout: post
title: JWT 기본과 Spring Boot 연동
summary: JWT 인증
author: devhtak
date: '2021-02-28 21:41:00 +0900'
category: Spring
---

#### JWT(JSON Web Token)

- JSON Web Token 이란
  - JSON 객체로 데이터를 안전하게 전송하기 위한 개방형 표준(RFC7519)이다.
  - 해당 정보는 디지털 서명이 되어 있어 신뢰할 수 있다.
    - 디지털 서명
  - HMAC 알고리즘 또는 RSA 또는 ECDSA를 사용하는 공개/개인키 쌍을 사용한다.
  
- JWT 사용 방식
  - Authorization
    - JWT를 사용하는 가장 일반적인 방식이다.
    - 클라이언트가 로그인 한 이후 JWT가 포함된 요청을 보내며, 토큰에 대한 허가를 바탕으로 서비스를 사용할 수 있게 한다.
    - 작은 overhead 장점으로 최근 SSO에서도 JWT를 많이 사용하고 있다.
  
  - Information Exchange
    - JWT는 두 개체 사이에서 안정성 있게 정보를 교환하기에 좋은 방법이다.
    - JWT는 개인/공개키를 사용한 인증을 사용하기 때문에 sender에 대한 검증이 가능하다.
    - header와 payload가 sign되어 있기 때문에 정보 조작에 대한 검증이 가능하다.

- JWT 구조
  - Header, Payload, Signature 구조로 이루어져 있으며 .으로 구분되어 있다.
    - XXXXX.YYYYY.ZZZZZ
  - Header
    - 헤더는 일반적으로 2가지로 구성되어 있다. 타입과 암호 알고리즘
    - 암호 알고리즘은 HMAC, SHA256, RSA 등을 사용한다.
      ```
      {
          "alg": "HS256",
          "typ": "JWT"
      }
      ```
  - Payload
    - 두 번째 부분은 payload 이며, claims를 포함하고 있다.
    - claims
      - Entity(일반적으론 User) 및 추가 데이터
      - (type)Registerd claims: 서비스에 필요한 정보들이 아닌, 토큰에 대한 정보를 담기 위해 이미 정해진 클레임, optional
      - (type)public claims: 충돌을 방지하기 위한 이름으로 URI 형식으로 짓습니다.
      - (type)private claims: 클라이언트와 서버간의 협의 하에 사용되는 클레임
    - 예제
      ```
      {
          "sub": "!234567890",
          "name": "John Doe",
          "admin": true
      }
      ```
  - Signature
    - Header의 인코딩 값과 Payload의 인코딩한 값을 주어진 비밀키로 해쉬하여 생성한다.
    - HMAC SHA256 알고리즘으로 생성한 Signature 예제
      ```
      HMACSHA256(
          base64UrlEncode(header) + "." +
          base64UrlEncode(payload), secret
      )
      ```
      
  - JWT는 Base64Url로 인코딩하여 형성한다.
- 출처: https://jwt.io/introduction/

#### Spring Boot에서 JWT 사용하기

- 소스 저장소 위치: 

- 의존성 추가
  - jjwt-api, jjwt-impl, jjwt-jackson
  ```
  <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api -->
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-api</artifactId>
      <version>0.11.2</version>
  </dependency>
  <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-impl -->
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-impl</artifactId>
      <version>0.11.2</version>
      <scope>runtime</scope>
  </dependency>
  <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-jackson -->
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-jackson</artifactId>
      <version>0.11.2</version>
      <scope>runtime</scope>
  </dependency>
  ```

