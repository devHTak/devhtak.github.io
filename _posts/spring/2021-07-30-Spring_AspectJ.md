---
layout: post
title: AspectJ
summary: Spring Basic
author: devhtak
date: '2021-07-30 21:41:00 +0900'
category: Spring
---

#### AOP 란?

- Aspect Oriented Programming의 약자로 관점 지향 프로그래밍
- 로직에 대해 핵심과 부가적인 관점으로 나누고 그 관점을 기준으로 모듈화한 것
  - 흩어진 관심사(부가)를 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하겠다는 것이 AOP의 취지
  
- 주요 개념
  - Aspect
    - 흩어진 관심사를 모듈화 한 것
  - Target
    - Aspect를 적용하는 곳(클래스 또는 메소드)
  - Advice
    - 실질적으로 어떤 일을 해야할 지에 대한 것으로 실질적인 부가기능을 담은 구현체 
  - JoinPoint
    - Advice가 적용될 위치, 끼어들 수 있는 지점. Target 실행 중 다양한 시점에 적용이 가능하다.
  - PointCut
    - JoinPoint의 상세한 스펙을 정의한 것
    - 'A란 타겟의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있다.
    
#### AspectJ 사용

- dependency 추가
  ```
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
  </dependency>
  ```
  - aspectjrt는 Rutntime 프로그램이다.
  - aspectjweaver는 aspect의 정보를 바탕으로 aspect를 구성한 코드를 생성하는 데 필요한 유틸리티 프로그램
  
- Aspect 정의
  - 흩어진 관심사에 대해 모아둔 클래스로 @Aspect 어노테이션을 활용하여 생성할 수 있다.
  
  ```java
  @Aspect
  public class AspectClass {
    // ...
  }
  ```
  
- Pointcut 정의
  - Pointcut은 Advice가 언제 실행될지를 지정하는 데 사용한다.
  - Pointcut 지정자
    - execution: 메소드 실행 결합점(join points)과 일치시키는데 사용된다.
      - 접근지시자 리턴타입 메소드명(아규먼트) 형식으로 작성      
      
    - within: 특정 타입에 속하는 결합점을 정의한다.
      - within(com.example.service.*): com.example.service 내의 모든 결합점
      - within(com.example.service..*): com.example.service 패키지 및 하위 패키지 내의 모든 결합점
      
    - this: 빈 참조가 주어진 타입의 인스턴스를 갖는 결합점을 정의한다.
      - this(com.example.service.AccountService): AccountService 인터페이스를 구현하는 프록시 개체의 모든 결합점
      
    - target: 대상 객체가 주어진 타입을 갖는 결합점을 정의한다.
      - target(com.example.service.AccountService): AccountService 인터페이스를 구현하는 대상 객체의 모든 결합점
      
    - args: 인자가 주어진 타입의 인스턴스인 결합점을 정의한다.
      - args(java.io.Serializable): 하나의 파라미터를 갖고 전달된 인자가 Serializable인 모든 결합점
    
    - @target: 수행중인 객체의 클래스가 주어진 타입의 어노테이션을 갖는 결합점을 정의한다.
      - @target(org.springframework.transaction.annotation.Transactional): 대상 객체가 @Transactional 어노테이션을 갖는 모든 결합점
    
    - @args: 전달된 인자의 런타입 타입이 주어진 타입의 어노테이션을 갖는 결합점을 정의한다.
      - @args(com.xyz.security.Classified): 단일 파라미터를 받고, 전달된 인자 타입이 @Classified 어노테이션을 갖는 모든 결합점    
    
    - @within: 주어진 어노테이션을 갖는 타입 내 결합점을 정의한다.
      - @within(org.springframework.transaction.annotation.Transactional): 대상 객체의 선언 타입이 @Transactional 어노테이션을 갖는 모든 결합점
    
    - @annotation: 결합점의 대상 객체가 주어진 어노테이션을 갖는 결합점을 정의한다.
      - @annotation(org.springframework.transaction.annotation.Transactional): 실행 메소드가 @Transactional 어노테이션을 갖는 모든 결합점

- Advice 정의

  - Advice는 Aspect의 실제 구현체로 포인트컷 표현식과 일치하는 결합점에 삽입되어 동작할 수 있는 코드이다.
  - Advice는 Pointcut과 결합하여 동작하는 시점에 따라 before, after, around, after throwing 타입으로 구분된다.

  - Before
    - Before advice는 타겟이 실행되기 전에 실행된다.
    - @Before 어노테이션을 사용한다.
    - 예제
      
      ```java
      @Aspect
      public class AspectExample {
          @Before("execution(* *(..))")
          public void beforeTargetMethod(JoinPoint thisJoinPoint) {
              Class clazz = thisJoinPoint.getTarget().getClass();
              String className = thisJoinPoint.getTarget().getClass().getSimpleName();
              String methodName = thisJoinPoint.getSignature().getName();
              System.out.println("AspectUsingAnnotation.beforeTargetMethod executed.");
              System.out.println(className + "." + methodName + " executed.");
          }
      }
      ```
      - 모든 패키지의 모든 메소드의 실행 이전에 실행된다.
      
  - After returning
    - After returing Advice는 정상적으로 메소드가 실행될 때 수행된다.
    - After returning Advice는 @AfterReturing 어노테이션을 사용한다.
    - 예제
    
      ```java
      @Aspect
      public class AspectExample {
        ..
        @AfterReturning(pointcut = "targetMethod()", returning = "retVal")
        public void afterReturningTargetMethod(JoinPoint thisJoinPoint, Object retVal) {
          System.out.println("AspectUsingAnnotation.afterReturningTargetMethod executed." + " return value is [" + retVal + "]");
        }
      }
      ```
      - 해당 결과 값을 retVal을 통해 받을 수 있다.
      
  - After throwing
    - After throwing 충고는 메소드가 수행 중 예외사항을 반환하고 종료하는 경우 수행된다.
    - After throwing 충고는 @AfterThrowing 어노테이션을 사용한다.
    - 예제
      
      ```java
      @Aspect
      public class AspectExample {
        @AfterThrowing(pointcut = "targetMethod()", throwing = "exception")
        public void afterThrowingTargetMethod(JoinPoint thisJoinPoint, Exception exception) throws Exception {
          System.out.println("AspectUsingAnnotation.afterThrowingTargetMethod executed.");
          System.out.println("에러가 발생했습니다.", exception);

          throw new BizException("에러가 발생했습니다.", exception);
        }
      }
      ```
      
      - 다음은 After throwing 충고를 사용하는 예제이다. afterThrowingTargetMethod() 충고는 targetMethod()로 정의된 포인트컷에서 예외가 발생한 후에 수행된다. 
      - targetMethod() 포인트컷에서 발생된 예외는 exception 변수에 저장되어 전달된다. 
      - 전달 받은 예외를 한번 더 감싸서 사용자가 쉽게 알아 볼 수 있도록 메시지를 설정하여 반환한다.


  - After (finally)
    - After (finally) advice는 메소드 수행 후 무조건 수행된다.
    - After (finally) advice는 @After 어노테이션을 사용한다. 
    - After advice는 정상 종료와 예외 발생 경우를 모두 처리해야 하는 경우에 사용된다. 
    - 리소스 해제와 같은 작업이 해당된다.
    - 예제
    
      ```java
      @Aspect
      public class AspectExample {
          ..
          @After("targetMethod()")
          public void afterTargetMethod(JoinPoint thisJoinPoint) {
              System.out.println("AspectUsingAnnotation.afterTargetMethod executed.");
          }
      }
      ```
      - afterTargetMethod() 충고는 targetMethod()로 정의된 포인트컷 이후에 수행된다.

  - Around
    - Around advice는 메소드 수행 전후에 수행된다.
    - Around 충고는 @Around 어노테이션을 사용한다.
    - 예시
      
      ```java
      @Aspect
      public class AspectExample {
        @Around("targetMethod()")
        public Object aroundTargetMethod(ProceedingJoinPoint thisJoinPoint) throws Throwable {
          System.out.println("AspectUsingAnnotation.aroundTargetMethod start.");
          long time1 = System.currentTimeMillis();
          Object retVal = thisJoinPoint.proceed();

          System.out.println("ProceedingJoinPoint executed. return value is [" + retVal + "]");
          retVal = retVal + "(modified)";
          System.out.println("return value modified to [" + retVal + "]");

          long time2 = System.currentTimeMillis();
          System.out.println("AspectUsingAnnotation.aroundTargetMethod end. Time(" + (time2 - time1) + ")");
          return retVal;
        }
      }
      ```
      - aroundTargetMethod() 충고는 파라미터로 ProceedingJoinPoint을 전달하며 proceed() 메소드 호출을 통해 대상 포인트컷을 실행한다.
      - 포인트컷 수행 결과값인 retVal을 Around 충고 내에서 변환하여 반환할 수 있음을 보여준다.
      
#### 출처

- AspectJ사용: https://araikuma.tistory.com/309
- AspectJ사용: https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj
- AOP: https://engkimbs.tistory.com/746
