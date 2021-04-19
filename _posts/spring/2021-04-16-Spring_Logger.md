---
layout: post
title: Spring Logger
summary: 김영한님 - 스프링 MVC - 백엔드 웹 개발 핵심 기술
author: devhtak
date: '2021-04-16 21:41:00 +0900'
category: Spring
---

#### Logging

- 로깅 라이브러리
  - 스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리(spring-boot-starter-logging)이 함께 포함된다.
  - 기본으로 다음 로깅 라이브러리를 사용한다.
    - SLF4J: http://www.slf4j.org
    - Logback: http://logback.qos.ch
    - 로그 라이브러리로 Logback, Log4J, Log4J2 등 수 많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제공하는 것이 SLF4J이다.
    - SLF4J는 인터페이스이고, 그 구현체로 Logback 제공

- 로그 선언
  ```java
  @Slf4j //lombok
  @RestController
  public class LogTestController {

      // private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "spring";

        System.out.println("name : " + name);

        log.info("info log={}", name);
        log.trace("info trace={}", name);
        log.debug("info debug={}", name);
        log.warn("info warn={}", name);
        log.error("info error={}", name);

        return "ok";
    }
  }
  ```
  - SLF4J 패키지에 있는 Logger, LoggerFactory 사용
  - 로그가 출력되는 포멧 확인
    - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스명, 로그 메시지
  - 로그 레벨
    - LEVEL: TRACE > DEBUG > INFO > WARN > ERROR
    - 대체로 개발 서버는 debug 출력, 운영 서버는 info 출력

- 로그레벨 설정: application.properties
  ```
  #전체 로그 레벨 설정(기본 info)
  logging.level.root=info
  
  #hello.springmvc 패키지와 그 하위 로그 레벨 설정  
  logging.level.hello.springmvc=debug
  ```
  
- 올바른 로그 사용법
  - log.debug("data="+data)
    - 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 
    - 결과적으로 문자 더하기 연산이 발생한다.
    - log.debug("data={}", data)로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

- 로그 사용시 장점
  - 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
  - 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
  - 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다. 
  - 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
  - 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.
    
- 참고
  - 스프링 부트가 제공하는 로그 기능
    - https://docs.spring.io/spring-boot/docs/current/reference/html/spring-bootfeatures.html#boot-features-logging

** 출처: 스프링 MVC - 백엔드 웹 개발 핵심 기술
