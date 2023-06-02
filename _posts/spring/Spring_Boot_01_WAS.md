---
layout: post
title: Spring Boot. Embedded WAS
summary: Spring DB
author: devhtak
date: '2023-06-02 21:41:00 +0900'
category: Spring
---

#### Spring Framework
- Java 표준 기술로 EJB가 있었지만, 너무 복잡했고 가격이 비싸고, 의존적 개발이 필요했다.
  - POJO 개발을 하고 싶다는 니즈가 생겼다.
- Spring Framework
  - 핵심 기술: DI, AOP, event
  - WEB: MVC, Webflux
  - Data: Transaction, JDBC, ORM, XML 지원
  - 기술 통합: 캐시, 이메일, 원격 접근, 스케줄링
  - TEST: jUnit - Spring 기반 테스트 지원
  - 다양한 라이브러리들을 편리하게 사용할 수 있도록 통합하여 생산성이 높아졌다
- Version
  - Spring Framework 1.0: XML
  - Spring Framework 2.0: XML 편의 기능 지원
  - Spring Framework 3.0: XML, 자바 코드 설정
  - Spring Framework 4.0: XML 자바 8
  - Spring Boot 1.0
  - Spring Framework 5.0, Spring Boot 2.0: Reactive Programming 지원
  - Spring Framework 6.0, Spring Boot 3.0: Java 17, Jakarta EE
- 스프링 설정 지옥
  - 무겁고 거대해지면서 많아진 라이브러리의 빈을 직접 등록해주어야 했다 -> 스프링 부트 등장

#### Spring Boot
- 시작을 위한 복잡한 설정과정을 스프링 부트가 해결해준다.
- 핵심 기능
  - WAS: Tomcat 같은 웹 서버를 내장하여 별도의 웹 서버를 설치하지 않아도 됨
  - Library 관리
    - 손쉬운 빌드 구성을 위한 starter 종속성 제공
    - 스프링과 외부 라이브러리의 버전을 자동 관리
  - 자동 구성: 프로젝트 시작에 필요한 스프링과 외부 라이브러리의 빈을 자동 등록
  - 외부 설정: 환경에 따라 달라져야 하는 외부 설정 공통화
  - Production 준비: 모니터링을 위한 메트릭, 상태 확인 기능 제공

#### Web Server와 Servlet Container
- 전통 방식
  - 서버에 WAS를 설치하고, WAR 파일을 WAS에 전달하여 배포하는 형식
  - WAS 연동하여 실행되도록 추가 설정 필요
- 최근 방식
  - Spring Boot가 내장 톰캣과 같은 WAS를 포함(Library 형태)
  - 코드를 작성하고 JAR를 필드한 다음 JAR을 실행하면 WAS도 함께 실행
- JAR vs WAR
  - JAR(Java Archive)
    - Java의 여러 클래스와 리소스를 묶어 JAR라고 하는 압축파일을 만들 수 있고, 해당 파일은 JVM에서 실행되거나 다른 곳에서 라이브러리로 사용될 수 있다.
    - 직접 실행하기 위해서 main() 메소드가 필요하고, MANIFEST.MF 파일에 실행할 메인 메서드가 있는 클래스를 지정해야 한다
  - WAR(Web Application Archive)
    - WAS에 배포할 때 사용하는 파일로 HTML와 같은 정적 리소스와 클래스 파일을 모두 함께 포함하기 때문에 구조가 더 복잡하다
  - WAR 구조
    - WEB-INF: 폴더 하위는 자바 클래스와 라이브러리 설정 정보가 들어간다
      - classes(실행 클래스 모음)
      - lib(라이브러리 모음)
      - web.xml(웹 서버 배치 설정 파일)
    - index.html: 정적 리소스
    - WEB-INF를 제외한 나머지 영역에는 HTML, CSS 같은 정적 리소스가 사용되는 영역이다.
- 서블릿 컨테이너 초기화 지원
  - Servlet Container 초기화
    - 필터, 서블릿 등록, 스프링 컨테이너 생성, Dispatcher Servlet 등록
    - WAS 제공 초기화 기능 및 web.xml 초기화
  - Spring MVC 초기화 과정
    - ServletContainerInitializer 인터페이스 구현한 서블릿 컨테이너 객체 생성, 그 안에서 서블릿 초기화 코드 구현
    - @HandlesTypes로 애플리케이션 초기화 작업할 인터페이스 명시
    - 위에서 등록한 인터페이스 구현체 작성
    - META-INF/services/packages.구현체 등록
    - 서버 기동시 초기화 작업 인터페이스를 구현한 객체들을 각각 실행
    - 구현체를 늘리기만 하면 확장에 연려 있는 구조
  - Spring MVC 초기화 분석
    - WebApplicationInitializer 인터페이스 하나로 애플리케이션 초기화가 가능한 이유
    - spring-web 라이브러리의 META-INF/services 디렉토리 하위 SpringServletContainerInitializer 등록
    - SpringServletContainerInitializer 클래스 위해 @HandleTypes(WebApplicationInitializer.class) 기재
    - onStartup 메소드 구현부에 webAppInitializerClasses 인터페이스를 구현한 구현체들을 리플렉션으로 생성한 뒤 실행

#### Spring Boot와 Embedded WAS
- 내장 톰캣 실행
  ```java
  Tomcat tomcat = new Tomcat();
  Connector connector = new Connector(); // 커넥터 사용하여 8080 port 연결
  connector.setPort(8080);
  tomcat.setConnector(connector);
  
  Context context = tomcat.addContext("", "/");
  tomcat.addServlet("", "helloServlet", new HttpServlet()); // 서블릿 등록
  context.addServletMappingDecoded("/hello-servlet", "helloServlet"); // 서블릿 경로 매핑
  tomcat.start(); // 톰캣 시적
  ```
- FatJar
  - jar 파일 내에 jar 파일을 둘 수 없기 때문에 FatJar를 활용하여 jar 파일 내에 class 들을 뽑아서 만들고자 하는 jar에 포함시키는 기술
  - Spring Boot를 통해 만든 JAR 내에 내장 톰캣 라이브러리를 사용할 수 있는 것도 이 FatJAR 때문이다
  - 단점으로는 압축을 풀기 때문에 어떤 라이브러리가 포함되었는지 알기 어렵고 동일한 FQCN에 대하여 구별하기 어렵다
- Executable JAR(실행 가능 JAR)
  - Spring Boot 는 이런 문제를 해결하기 위해 jar 내부에 jar를 포함할 수 있는 특별한 구조의 jar를 만든다.
  - 동시에 만든 jar를 내부 jar를 포함애 실행할 수 있게 했다.
  - FatJAR 에 단점 해결
    - jar 내부에 jar를 포함하기 때문에 어떤 라이브러리가 포함되었는 지 알 수 있고, 동일한 경로의 파일도 둘 다 인식할 수 있다
