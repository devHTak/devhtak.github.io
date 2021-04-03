---
layout: post
title: 더 자바 테스트6. ArchUnit
summary: Architecture Test
author: devhtak
date: '2021-04-03 21:41:00 +0900'
category: Test
---

#### ArchUnit 소개

- 애플리케이션의 아키텍처를 테스트할 수 있는 오픈 소스 라이브러리로, 패키지, 클래스, 레이어, 슬라이스 간의 의존성을 확인할 수 있는 기능 제공

- 아키텍처 테스트 유즈 케이스
  - A라는 패키지가 B 또는 C, D 패키지에서만 사용되고 있는지 확인 가능
  - *Service라는 이름의 클래스들이 *Controller 또는 *Service라는 이름의 클래스에서만 참조하고 있는지 확인
  - *Service라는 이름의 클래스들이 ..service.. 라는 패키지에 들어있는지 확인
  - A라는 애노테이션을 선언한 메소드만 특정 패키지 또는 특정 애노테이션을 가진 클래스를 호출하고 있는지 확인
  - 특정한 스타일의 아키텍처를 따르고 있는지 확인
  
- 참고
  - https://www.archunit.org/
  - https://blogs.oracle.com/javamagazine/unit-test-your-architecture-with-archunit
  - https://www.archunit.org/userguide/html/000_Index.html
  - moduliths: https://github.com/odrotbohm/moduliths
  
#### ArchUnit 설치

- 참고: https://www.archunit.org/userguide/html/000_Index.html#_junit_5
- dependency
  ```
  <dependency>
      <groupId>com.tngtech.archunit</groupId>
      <artifactId>archunit-junit5-engine</artifactId>
      <version>0.12.0</version>
      <scope>test</scope>
  </dependency>
  ```

- 주요 사용법
  - 특정 패키지에 해당하는 클래스를 (바이트코드를 통해)읽어들이고
  - 확인할 규칙을 정의하고
  - 읽어들인 클래스들이 그 규칙에 잘 따르는지 확인
  
  ```java
  @Test
  public void ServiceShouldOnlyBeAccessedByControllers() {
      JavaClasses importedClasses = new ClassFileImporter().importPackages("com.test.myapp");
      
      ArchRule archRule = classes()
          .that().resideInAPackage("..service..")
          .should().onlyBeAccessed().byAnyPackage("..controller..", "..service..");
          
      archRule.check(importedClasses);
  }  
  ```

- JUnit 5 확장팩 제공
  - @AnalyzeClasses: 클래스를 읽어들여서 확인할 패키지 설정
  - @ArchTest: 확인할 규칙 정의

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
