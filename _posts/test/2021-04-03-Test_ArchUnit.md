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
  - \*Service라는 이름의 클래스들이 \*Controller 또는 \*Service라는 이름의 클래스에서만 참조하고 있는지 확인
  - \*Service라는 이름의 클래스들이 ..service.. 라는 패키지에 들어있는지 확인
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
	void services_should_only_be_accessed_by_controllers() {
      // 특정 패키지에 해당 클래스를 바이트 코드를 통해 읽어들이고
      JavaClasses importedClasses = new ClassFileImporter()
          .importPackages("com.study.study");

      // 확인할 규칙 정의
      ArchRule archRule = ArchRuleDefinition.classes()
          .that().resideInAPackage("..service..")
          .should().onlyBeAccessed().byAnyPackage("..controller..", "..service..");

      // 읽어들인 클래스들이 그 규칙을 잘 따르고 있는 지 확인
      archRule.check(importedClasses);
	}  
  ```

- JUnit 5 확장팩 제공
  - @AnalyzeClasses: 클래스를 읽어들여서 확인할 패키지 설정
  - @ArchTest: 확인할 규칙 정의

#### 패치지 의존성 확인하기

- 예제
  - Study -> Member
  - Study -> Domain
  - Member -> Domain

- 실제로 그런지 확인하려면
  - ..domain.. 패키지에 있는 클래스는 ..study.., ..member.., ..domain..에서 참조 가능
  - ..member.. 패키지에 있는 클래스는 ..study..와 ..member..에서만 참조 가능
    - 반대로, ..domain.. 패키지는 ..member.. 패키지를 참조하지 못한다.
  - ..study.. 패키지에 있는 클래스는 ..study.. 에서만 참조 가능
  - 순환 참조가 없어야 한다.

  ```java
  @Test
	void packageDependencyTest() {
      JavaClasses classes = new ClassFileImporter().importPackages("com.study");

      // ..domain.. 패키지에 있는 클래스는 ..study.., ..member.., ..domain..에서 참조 가능
      ArchRule domainPackageRule = ArchRuleDefinition.classes()
          .that().resideInAPackage("..domain..")
          .should().onlyBeAccessed().byAnyPackage("..study..", "..member..", "..domain..");		
      domainPackageRule.check(classes);

      // ..domain.. 패키지는 ..member.. 패키지를 참조하지 못한다.
      ArchRule memberPackageRule = ArchRuleDefinition.noClasses()
          .that().resideInAPackage("..domain..")
          .should().accessClassesThat().resideInAPackage("..member..");
      memberPackageRule.check(classes);

      // ..study.. 패키지에 있는 클래스는 ..study.. 에서만 참조 가능
      ArchRule studyPackageRule = ArchRuleDefinition.noClasses()
          .that().resideOutsideOfPackage("..study..")
          .should().accessClassesThat().resideInAPackage("..study..");
      studyPackageRule.check(classes);

      // 순환 참조가 없어야 한다.
      ArchRule freeOfCycles = SlicesRuleDefinition.slices().matching("com.study.(*)..")
          .should().beFreeOfCycles();
      freeOfCycles.check(classes);		
  }
  ```

#### JUnit 5 연동하기

- @AnalyzeClasses: 클래스를 읽어들여서 확인할 패키지 설정
- @ArchTest: 확인할 규칙 정의

```java
// Application.class가 있는 패키지 선택
@AnalyzeClasses(packagesOf = Application.class)
public class ArchTests {

    // ..domain.. 패키지에 있는 클래스는 ..study.., ..member.., ..domain..에서 참조 가능
    @ArchTest
    ArchRule domainPackageRule = ArchRuleDefinition.classes()
      .that().resideInAPackage("..domain..")
      .should().onlyBeAccessed().byAnyPackage("..study..", "..member..", "..domain..");

    // ..domain.. 패키지는 ..member.. 패키지를 참조하지 못한다.
    @ArchTest
    ArchRule memberPackageRule = ArchRuleDefinition.noClasses()
      .that().resideInAPackage("..domain..")
      .should().accessClassesThat().resideInAPackage("..member..");

    // ..study.. 패키지에 있는 클래스는 ..study.. 에서만 참조 가능
    @ArchTest
    ArchRule studyPackageRule = ArchRuleDefinition.noClasses()
      .that().resideOutsideOfPackage("..study..")
      .should().accessClassesThat().resideInAPackage("..study..");

    // 순환 참조가 없어야 한다.
    @ArchTest
    ArchRule freeOfCycles = SlicesRuleDefinition.slices().matching("com.study.(*)..")
      .should().beFreeOfCycles();	
}
```

#### 클래스 의존성 확인하기

- Study 패키지 안에 확인하려는 클래스 의존성
  - StudyController -> StudyRepository
  - StudyController -> StudyService
  - StudyService -> StudyRepository

- 테스트 할 내용
  - StudyController는 StudyService와 StudyRepository를 사용할 수 있다.
  - Study\*로 시작하는 클래스는 ..study.. 패키지에 있어야 한다.
    - domain에 포함되어 있는, Entity, Enum 은 제외
  - StudyRepository는 StudyService와 StudyController를 사용할 수 없다.

  - 코드
    ```java    
    @AnalyzeClasses(packagesOf = Application.class)
    public class ArchStudyTests {

        // StudyController는 StudyService와 StudyRepository를 사용할 수 있다.
        @ArchTest
        ArchRule controllerClassRule = ArchRuleDefinition.classes()
            .that().haveSimpleNameEndingWith("Controller")
            .should().accessClassesThat().haveSimpleNameEndingWith("Repository")
            .orShould().accessClassesThat().haveSimpleNameEndingWith("Service");

        // Study*로 시작하는 클래스는 ..study.. 패키지에 있어야 한다.
        @ArchTest
        ArchRule studyClassPackageRule = ArchRuleDefinition.classes()
            .that().haveSimpleNameStartingWith("Study")
            .and().areNotEnums()
            .and().areNotAnnotatedWith(Entity.class)
            .should().resideInAPackage("..study..");

        // StudyRepository는 StudyService와 StudyController를 사용할 수 없다.
        @ArchTest
        ArchRule repositoryClassRule = ArchRuleDefinition.noClasses()
            .that().haveSimpleNameEndingWith("Repository")
            .should().accessClassesThat().haveSimpleNameEndingWith("Service")
            .orShould().accessClassesThat().haveSimpleNameEndingWith("Controller");
    }
    ```

##### 다양한 방법 정리

- Selenium WebDriver
  - 웹 브라우저 기반 자동화된 테스트 작성에 사용할 수 있는 툴
  - https://www.selenium.dev/projects/

- DBUnit
  - 데이터베이스에 데이터를 CVS, Excel 등으로 넣어주는 툴
  - http://dbunit.sourceforge.net/

- REST Assured
  - REST API 테스트 라이브러리
  - https://rest-assured.io/

- Cucumber
  - BDD를 지원하는 테스트 라이브러리.
  - https://cucumber.io/


** 참조: 백기선님 더 자바 애플리케이션을 테스트하는 다양한 방법 강의
