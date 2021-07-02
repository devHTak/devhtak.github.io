---
layout: post
title: Proxy 패턴과 Decorator 패턴
summary: Java Study
author: devhtak
date: '2021-07-02 21:41:00 +0900'
category: Java Study
---

#### Proxy Pattern

![image](https://user-images.githubusercontent.com/42403023/124233346-b7f23280-db4d-11eb-8bdb-28bd07ed9d47.png)

*** 이미지 출처: https://sourcemaking.com/design_patterns/proxy

- Proxy 객체에는 RealSubject에 대한 레퍼런스가 들어있다.
- Proxy, RealSubject는 동일하게 Subject interface에 대한 구현체이기 때문에 RealSubject가 들어갈 자리에 Proxy를 사용할 수 있다.
- 그래서, 클라이언트와 RealSubject 중간에 Proxy를 두고 Proxy를 통해서 데이터를 주고 받는다.

- 프록시 패턴의 종류
  
  |종류|설명|
  |---|---|
  |원격 프록시|원격 객체에 대한 접근 제어 가능|
  |Virtual Proxy|객체의 생성비용이 많이 들어 미리 생성하기 힘든 객체에 대한 접근 및 생성시점 등을 제어|
  |Protection Proxy|객체에 따른 접근 권한을 제어해야하는 객체에 대한 접근을 제어할 수 있음|
  |방화벽 프록시|일련의 네트워크 자원에 대한 접근을 제어함으로써 주 객체를 '나쁜' 클라이언트들로부터 보호하는 역할|
  |Smart Reference Proxy|주 객체가 참조될 때마다 추가 행동을 제공 ex) 객체 참조에 대한 선 작업, 후 작업 등|
  |Caching Proxy|비용이 많이 드는 작업의 결과를 임시로 저장, 추후 여러 클라이언트에 저장된 결과를 실제 작업처리 대신 보여주고 자원을 절약하는 역할|
  |Synchronization Proxy|여러 스레드에서 주 객체에 접근하는 경우에 안전하게 작업을 처리 가능. 주로 분산 환경에서 일련의 객체에 대한 동기화 된 접근을 제어해주는 자바 스페이스에서 사용|
  |Complexity Hiding Proxy|복잡한 클래스들의 집합에 대한 접근을 제어 및 복잡도를 숨김|
  |Copy-On-Write Proxy|클라이언트에서 필요로 할 때까지 객체가 복사되는 것을 지연시킴으로써 객체의 복사를 제어|
  
#### Decorator Pattern

![image](https://user-images.githubusercontent.com/42403023/124234273-c856dd00-db4e-11eb-9588-5c7657c8107c.png)

*** 이미지 출처: https://sourcemaking.com/design_patterns/decorator

- 객체의 결합을 통해 기능을 동적으로 유연하게 확장할 수 있게 해주는 패턴
  - 기본 기능에 추가할 수 있는 기능의 종류가 많은 경우에 각 추가 기능을 Decorator 클래스로 정의한 후 필요한 Decorator 객체를 조합함으로써 추가 기능의 조합을 설계하는 방식

- 구조 페턴의 하나로 기본 기능에 추가할 수 있는 많은 종류의 부가 기능에서 파생되는 다양한 조합을 동적으로 구현할 수 있는 패턴이다.
  - 구조(Structural) 패턴
    - 클래스나 객체를 조합해 더 큰 구조를 만드는 패턴
    - 예를 들어 서로 다른 인터페이스를 지닌 2개의 객체를 묶어 단일 인터페이스를 제공하거나 객체들을 서로 묶어 새로운 기능을 제공하는 패턴이다.
  - 합성 관계
    - 생성자에서 필드에 대한 객체를 생성하는 경우 전체 객체의 라이프타임과 부분 객체의 라이프 타임은 의존적이다.
    - 즉, 전체 객체가 없어지면 부분 객체도 없어진다.
    
- 역할 수행 작업
  - Interface
    - 기본 기능을 뜻하는 CoreFunctionality와 추가 기능을 뜻하는 Decorator의 공통 기능을 정의
    - 즉, 클라이언트는 Interface를 통해 실제 객체를 사용함
  - CoreFunctionality
    - 기본 기능을 구현하는 클래스
  - OptionalWrapper
    - 많은 수가 존재하는 구체적인 Decorator의 공통 기능을 제공
  - OptionalOne, OptionalTwo, OptionalThree
    - Decorator의 하위 클래스로 기본 기능에 추가되는 개별적인 기능을 뜻함
    - ConcreteDecorator 클래스는 ConcreteComponent 객체에 대한 참조가 필요한데, 이는 Decorator 클래스에서 Component 클래스로의 ‘합성(composition) 관계’를 통해 표현됨

#### Proxy vs Decorator

- 공통점
  - class의 구조가 비슷하다. 
    - 둘다 동일한 Interface를 구현합니다.
    - Wrapper Class와 real class의 관계가 aggregation 즉, has A 관계를 띄고 있다.

- 차이점
  - 프록시 패턴에서는 Wrapper Class와 Real Class의 관계가 컴파일타임에 정해집니다. 반면 데코레이터 패턴에서는 런타임에 정해진다.
  - 프록시 패턴은 Real Class의 접근에 대한 제어가 목적, 데코레이터 패턴은 Real Class의 기능에 다른 기능을 추가하는 것이 목적.

#### 출처

- https://sabarada.tistory.com/60
- https://developside.tistory.com/80
- https://gmlwjd9405.github.io/2018/07/09/decorator-pattern.html
