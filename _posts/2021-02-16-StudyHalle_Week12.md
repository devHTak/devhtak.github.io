---
layout: post
title: 스터디 할래 12. Annotation
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-02-16 21:41:00 +0900'
category: Java Study
---

#### 학습할 것

- 애노테이션 정의하는 방법
- @retention
- @target
- @documented
- 애노테이션 프로세서

#### 애노테이션 정의하는 방법

- Annotation
  - @로 시작하며 Java 5부터 등장했다.
  - 메서드/클래스 등에 의미를 단순히 컴파일러에게 알려주기 위한 표식이 아닌 컴파일타임 이나 런타임에 해석될 수 있다.
  
  - 장점
    - 기존의 자바는 선언적 프로그래밍방식으로 개발을 하면서 각 계층별 설정 데이터들을 XML에 명시했었는데 서비스의 규모가 클 수록 설정양이 많아지고 도메인 처리의 데이터들이 분산되어 있어 수정이 힘들었다.
    - 어노테이션이 등장하면서 데이터 유효성 검사 등 직접 클래스에 명시해 줄 수 있게되어 수정이 필요할때 쉽게 파악할 수 있게 되었고 어토테이션의 재사용도 가능해졌다.
  
  - 용도
    - 크게 문서화, 컴파일러 체크, 코드 분석과 자동 생성,런타임 프로세싱 용도로 사용될 수 있다.
    - 컴파일 타임에 에러를 발생 시켜 경고하는 목적으로 사용될 수 있고 문서화는 컴파일 시 어노테이션이 붙은 데이터를 수집하여 가능하지만 가장 비중이 낮은 사용법이다.
    - 유효성 검사와 같은 메타데이터로써 사용되고 reflection을 이용하여 특정 클래스를 주입할 수도 있다.
    - AOP(관점 지향 프로그래밍)을 쉽게 구성할 수 있게 해준다.
    - Meta Data
      - “어떤 목적을 가지고 만들어진 데이터” -Karen Coyle-
      - 한마디로 어떤 데이터를 설명해주는 데이터
    - Reflection
      - 반사,투영이 라는 뜻으로 객체를 통해 클래스의 정보를 분석해내는 기법
      - ClassName, SuperClass, Constructors, Methods, Fields, Annotations …
      
  - 분류
    - Maker 어노테이션 : 멤버 변수가 없고 컴파일러에게 의미를 전달하기 위한 표식으로 사용되는 어노테이션 (ex. @Override )
    - Single-value 어노테이션 : 멤버로 단일변수를 갖고 데이터를 전달할 수 있는 어노테이션
    - Full 어노테이션 : 둘 이상의 변수를 갖는 어노테이션으로 데이터를 키 = 값형태로 전달한다.
      
- Annotation 사용 방법
  ```java
  public @interface AnnotationEx {
    // ...
  }
  ```
  
  - 규칙
   - 요소의 타입은 기본형, String, enum, 어노테이션, Class만 허용된다.
   - ()안에 매개변수는 선언할 수 없다.
   - 예외를 선언할 수는 없다.
   - 요소를 타입 매개변수로 정의 할 수 없다.
 
- 표준 Annotaition
  - @Override
    - 오버라이드를 할 때 사용되며, 메소드가 오버라이드됐는지를 알려준다.
  - @Deprecated
    - Java 버전이 올라가면서 더 이상 사용되지 않는 것 대상에 붙여진다.
  - @FunctionalInterface
    - Lambda로 작성할 수 있도록 함수형으로 사용할 수 있는 인터페이스앞에 작성한다.
    - 인터페이스 내에 추상메소드가 2개 이상이면 컴파일 오류가 발생한다.
    ```java
    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    public @interface FunctionalInterface {}
    ```
  - @SusspressWarnings
    - 컵팡딜러가 보여주는 경고 메시지를 보이지 않게 해준다.
  

#### Meta Annotation
- @retention
  - 어노테이션이 유지되는 기간(Life Time)을 설정하는 어노테이션
  - 설정 값
    - SOURCE : 소스파일에만 존재하고, 클래스파일에는 존재x, 컴파일러에 의해 버려진다. 바이트코드에 남아있지 않다.
    - CLASS : 클래스파일에는 존재하지만 런타임 시에 유지할 필요 없다는 것을 알리고 이 값이 default이다.
    - RUNTIME : 클래스파일에도 존재하고 런타임애 VM에 의해 유지되어 리플랙션을 통해 클래스 파일의 정보를 읽어 처리 가능하다.

- @target
  - 어노테이션이 적용가능한 대상(동작 대상)을 지정한다.
  - 만약 다른 타입이 온다면 컴파일 에러를 띄운다.
  - 아래와 같은 ElmentType이라는 enum을 통해 지정한다. ( @Target(ElemntType.~)와 같이 사용 )
  - 설정 값
    - TYPE : Class, Interface(어노테이션 타입 포함), enum, jdk14에 생긴 record
    - FIELD : 필드 값(프로퍼티), enum 상수값
    - METHOD : 메서드
    - PARAMETER : 메서드 파라미터 (매개 변수)
    - CONSTRUCTOR : 생성자
    - LOCAL_VARIABLE : 지역 변수
    - ANNOTATION_TYPE : 어노테이션
    - PACKAGE : 자바 패키지
    - TYPE_PARAMETER : 타입 매개 변수(1.8이후)
    - TYPE_USE : 타입 사용 (1.9이후)
    - MODULE : 모듈(1.8이후)
    
- @documented
  - 어노테이션의 정보가 javadoc의 문서에 포함되도록 하는 어노테이션

- @Inherited
  - 자식 클래스에게도 어노테이션이 상속되도록 하는 어노테이션

- @Repeatable
  - 어노테이션을 반복적으로 선언할 수 있게 하는 어노테이션

#### 애노테이션 프로세서

- 런타임시에 리플랙션을 사용하는 어노테이션과는 달리 컴파일 타임에 이루어진다.
- 컴파일 타임에 어노테이션들을 프로세싱하는 javac에 속한 빌드 툴로 어노테이션의 소스코드를 분석하고 처리하기 위해 사용되는 훅이다.
- 보일러플레이트 코드를 제거하는데 도움이 된다.
  - AbstractProcessor를 implements하여 구현체를 만들 수 있으며 Lombok의 @Getter, @Setter와 같은 어노테이션을 이용하는 것만으로도 컴파일 타임에 알아서 getter/setter를 만들어주는 방식으로 보일러플레이트 코드 제거
  
