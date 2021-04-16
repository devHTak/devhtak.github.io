---
layout: post
title: Spring Logger
summary: 김영한님 - 스프링 MVC - 백엔드 웹 개발 핵심 기술
author: devhtak
date: '2021-04-16 21:41:00 +0900'
category: Spring
---

#### Logging

- 운영환경에서 sysout 대신 로깅을 사용한다.
  - sysout에 경우 레벨에 상관없이 모두 남기 때문에, 대량의 요청이 오면 실제 필요한 정보만 남기기 어렵다.

- 로깅 라이브러리
  - 스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리(spring-boot-starter-logging)이 함께 포함된다.
  - 기본으로 다음 로깅 라이브러리를 사용한다.
    - SLF4j: http://www.slf4j.org
    - 
