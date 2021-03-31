---
layout: post
title: 스터디 할래 1.  JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가.
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-07 23:41:00 +0900'
category: Java Study
---

### JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가.

#### 목표

- 자바 소스 파일(.java)을 JVM으로 실행하는 과정 이해하기.

#### 학습할 내용

- JVM이란 무엇인가
- 컴파일 하는 방법
- 실행하는 방법
- 바이트코드란 무엇인가
- JIT 컴파일러란 무엇이며 어떻게 동작하는지
- JVM 구성 요소
- JDK와 JRE의 차이

#### JVM이란 무엇인가

- JAVA Virtual Machine의 약자로, 자바 바이트코드를 실행할 수 있는 주체
- 자바 바이트 코드(.class) 파일을 OS에 특화된 코드로 변환(인터프리터와 JIT 컴파일러)하여 실행

- JVM 특성
  - JAVA와 OS 사이에서 중개자 역할
    - OS에 구애받지 않고 사용 가능
  - Stack 기반의 가상머신
    - 데이터 흐름 분석(data flow analysis)에 기반한 자바 바이트코드 검증기(verifier)를 통해 Stack overflow 등이 발생한다.
    - 명령어 피연산자의 타입 규칙 위반, 필드 접근 규칙 위반, 지역 변수의 초기화 전 사용 등 많은 문제를 실행 전에 검증하여 실행 시 안전을 보장하고 별도의 부담을 줄여줌
  - 메모리관리, Garbage Collection을 수행
  
- 자바 프로그램 실행 과정
  - 프로그램이 실행되면 JVM은 OS로부터 메모리를 할당 받는다.
    - JVM은 이 메모리를 용도에 따라 여러 영역으로 나누어 관리
  - 자바 컴파일러(javac)가 자바 소스 코드(.java)를 읽어들여 자바 바이트코드(.class)로 변환
  - class loader를 통해 class파일들을 JVM으로 로딩
  - 로딩된 class 파일들은 execution engine을 통해 해석
  - 해석된 바이트코드는 Runtime Data Areas 에 배치되어 실질적인 수행이 이뤄진다.
  - 이러한 실행 과정 속에서 JVM은 필요에 따라 Thread Synchonization과 GC같은 관리 작업 수행

- 출처: 
  - JVM: 위키백과, https://asfirstalways.tistory.com/158  
  
#### 컴파일/실행 하는 방법

```
$ javac /src/.../file.java
$ java file
```
- javac : 컴파일러 실행, JDK 안에 존재
- java: 클래스 파일 실행, JRE 안에 존재

- 만약 JAVA 1.11 버전으로 컴파일 한 클래스 파일을 JAVA 1.8로 실행한다면 ? 실패... UnsupportedClassVersionError 발생
- 만약 JAVA 1.8로 컴파일 한 클래스 파일을 1.11로 실행한다면 ? 가능!!
- 그래서 컴파일러 옵션이 존재한다.

#### 바이트코드란 무엇인가

- 자바 바이트코드는 JVM이 실행하는 명령어의 형태
- 각각의 바이트코드는 1바이트로 구성되지만 몇 개의 파라미터가 사용되는 경우가 있어 총 몇 바이트로 구성되는 경우가 있다.
- 역컴파일러 / 디컴파일러
  - 컴파일러는 기계어(비트 단위)가 아닌 중간 단계인 바이트 코드 형태로 변환
  - 역컴파일러 / 디컴파일러는 실행코드에서 소스코드를 역으로 추출
- 예시
  - 자바 코드
    ```java
    for (int i = 2; i < 1000; i++) {
        for (int j = 2; j < i; j++) {
            if (i % j == 0)
                continue outer;
        }
        System.out.println (i);
    }
    ```
  - 바이트코드
    ```
    0:   iconst_2
    1:   istore_1
    2:   iload_1
    3:   sipush  1000
    6:   if_icmpge       44
    9:   iconst_2
    10:  istore_2
    11:  iload_2
    12:  iload_1
    13:  if_icmpge       31
    16:  iload_1
    17:  iload_2
    18:  irem
    19:  ifne    25
    22:  goto    38
    25:  iinc    2, 1
    28:  goto    11
    31:  getstatic       #84; // Field java/lang/System.out:Ljava/io/PrintStream;
    34:  iload_1
    35:  invokevirtual   #85; // Method java/io/PrintStream.println:(I)V
    38:  iinc    1, 1
    41:  goto    2
    44:  return
    ```
    
#### JIT 컴파일러란 무엇이며 어떻게 동작하는지

- JIT 컴파일러 
  - Just-in time compiler 또는 동적 번역(dynamic translation)은 프로그램을 실행하는 시점에 기계어로 번역하는 컴파일 기법
  - 정적 컴파일러 vs Interpreter
    - 인터프리터 방식은 실행 중 프로그래밍 언어를 읽어가면서 해당 기능에 대응하는 기계어 코드를 실행한다.
    - 정적 컴파일러는 실행하기 전에 프로그램 코드를 기계어로 번역한다.
    - JIT 컴파일러는 두가지 방식을 혼합한 방식으로 생각할 수 있다
      - 실행 시점에서 인터프리트 방식으로 기계어 코드를 생성하면서 그 코드를 캐싱하여, 같은 함수가 반복 호출될 때 매번 기계어 코드를 생성하는 것을 방지한다.\
  - JIT 컴파일러는 JRE안에 있다.
      
- 동작
  - HelloWorld.java 소스코드를 작성한 후 컴파일러로 바이트코드 변환(java -> class: 컴파일러)
  - JVM에서 각 운영 체제에 맞는 기계어로 번역해 전달
  
- 출처: 위키백과
  
#### JVM 구성 요소

- Class Loader, Execution Engine, Runtime Data Area(memory), Native로 구성된다.

- Class Loader
  - .class 에서 바이트코드를 읽고 메모리에 저장
  - 로딩, 링크, 초기화 순서로 진행된다.    
    - 로딩
      - 클래스를 읽어오는 과정
      - 클래스 로더가 바이트코드를 읽고 그 내용에 따라 적절한 바이너리 데이터(기계어)를 만들어 Method영역(메모리)에 저장한다. 
      - Method영역에 저장되는 것
        - Full Qualified Class Name (ex. org.springframework.boot.SpringApplication)
        - 클래스, 인터페이스, Enum
        - 메서드와 변수
      - 저장이 끝나면, 해당 Class의 Class 객체를 만들어 힙 영역에 저장한다.
    - 링크: Verify, Prepare, Resolve 세 단계로 나눠진다.
      - 레퍼런스를 연결하는 과정
      - verify(검증): 혹시 모를 수정에 대비하여 클래스 파일 형식이 유효한지 체크
      - prepare(준비): 클래스 변수와 기본값에 필요한 메모리 준비
      - resolve(해결): symbolic memory reference를 메서드 영역에 있는 실제 레퍼런스로 교체
        - symbolic reference: 실제 레퍼런스를 가리키고 있지 않은 상태, 논리적인 레퍼런스이다.
    - 초기화
      - static 값들 초기화 및 변수에 할당
      - static 변수의 값을 할당, static 블럭은 이 때 실행된다.
  - 클래스 파일을 적재하는 역할을 한다. 즉, 컴파일러로 바이트코드를 생성하면 해당 바이트코드를 클래스 로더가 메모리에 적재한다.
  - 기본적인 세가지 클래스 로더가 제공
    - Bootstrap ClassLoader: JAVA_HOME\lib에 있는 코어 자바 API를 제공, 최상위 우선순위를 가진 클래스 로더, Native로 구현됨
    - Platform ClassLoader: JAVA_HOME\lib\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
    - Application ClassLoader: Application ClassPath, 즉, 애플리케이션 실행할 때 주는 -classpath 또는 java.class.path 환경 변수의 값에 해당하는 위치에서 클래스를 읽는다.
    - 로딩 순서
      - Bootstrap ClassLoader -> Platform ClassLoader -> Application ClassLoader 순서로 찾는다.
      - 상위 ClassLoader에 없는 경우 하위 ClassLoader를 찾게 되며 보통 90% Application ClassLoader에 있다.
      - 찾지 못하는 경우 ClassNotFoundException이 발생한다.
    
- Execution Engine
  - Class Loader에 의해 Runtime Data Area에 적재된 클래스(바이트코드)들을 컴퓨터가 이해할 수 있는 기계어로 변경해 명령어 단위로 실행하는 역할
  - JIT Compiler
    - 인터프리터 효율을 높이기 위해, 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 모두 네이티브 코드로 바꿔둔다.
    - 그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.
  - GC(Garbage Collector): 더 이상 참조하지 않는 객체를 모아서 정리
    - 힙 영역에 생성된 객체들 중 참조되지 않는 객체들을 메모리에서 제거하는 역할
    - GC가 사용되는 동안에는 GC에 스레드가 아닌 다른 모든 스레드는 일시정지된다.
    
- Runtime Data Area(메모리 영역)
  - 메모리 영역에는 클래스 수준의 정보 (클래스 이름, 부모 클래스 이름, 메서드, 변수) 저장, 공유 자원이다.
  - Method(static) 영역
    - JVM에서 읽어들인 클래스와 인터페이스에 대한 런타임 상수 풀, 메서드와 필드, static 변수, 메서드 바이트 코드 등을 보관
  - Runtime Constant Pool 영역
    - Method 영역에 포함되지만 독자적 중요성을 띈다. 
    - 클래스 파일, constant_pool 테이블에 해당하는 영역으로 클래스와 인터페이스 상서, 메서드와 필드에 대한 모든 레퍼런스를 저장
    - JVM은 런타임 상수 풀을 통해 해당 메서드나 필드의 실제 메모리 상 주소를 찾아 참조한다.
  - Heap Area
    - 프로그램 상에서 데이터를 저장하기 위해 런타임 시 동적으로 할당하여 사용하는 메모리 영역
    - new 연산자를 통해 생성한 객체 또는 인스턴스와 배열 저장
    - JVM이 관리
  - Mehtod 영역, Runtime Constant Pool 영역은 JVM 시작 시 생성하여 프로그램 종료 시까지 유지되며 명시적으로 null 선언 시 GC 대상이 된다.
  - Heap 영역은 객체가 더 이상 쓰지 않거나, null 선언 시 GC 대상이 된다.
  - Mehtod 영역, Runtime Constant Pool 영역, Heap 영역은 모든 스레드에서 공유한다.
  - Stack 영역
    - FILO 구조
    - 메서드 호출 시 생성되는 스레드 수행 정보를 기록하는 frame 저장, 메서드 정보, 지역변수, 매개변수, 연산 중 발생하는 임시 데이터 저장
    - 사용 기간은 블록 또는 메서드가 끝날 때까지 유지된다.
  - PC Register
    - 현재 실행 중인 JVM 주소를 가지고 있다.
  - Native Method Stack Area
    - Native code를 위한 메모리(C/C++ 코드 수행)
    
- JNI(Java Native Interface)
  - 자바 애플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법 제공
  - Native 키워드를 사용한 메서드 호출
  
- 네이티브 메소드 라이브러리
  - C, C++로 작성된 
  
#### JDK와 JRE의 차이

- JDK
  - Java Development Kit로 JRE + Development Kit(개발에 필요한 툴)로 이루어져 있다.
  - 소스 코드를 작성할 때 사용하는 자바 언어는 플랫폼에 독립적
  - 오라클은 자바 11부터 JDK 만 제공하며 JRE를 따로 제공하고 있지 않다.
  
- JRE
  - Java Runtime Environment의 약자로 JVM + library로 이루어져 있다
  - 자바 애플리케이션을 실행할 수 있도록 구성한 배포판
  - JVM과 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 가지고 있다.
  - 개발 관련 도구는 포함하지 않는다(JDK에서 제공)
    
- JVM, JRE, JDK 출처: https://medium.com/webeveloper/jvm-java-virtual-machine-architecture-94b914e93d86
- 백기선님 인프런 강좌 \[더 자바, 코드를 조작하는 방법] 참고
