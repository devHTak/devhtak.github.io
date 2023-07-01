---
layout: post
title: Spring Document
summary: Spring Document
author: devhtak
date: '2022-05-10 21:41:00 +0900'
category: Spring
---
#### Spring AOT(Ahead of Time)

#### JVM의 JIT Compiler
- Java 파일 -> (컴파일 타임) -> Class 파일(바이트 코드) -> (런타임) -> 기계어로 변환되어 JVM 상에서 실행하게 된다.
- JVM에서 바이트 코드를 기계어로 번역하는 컴파일러가 바로 JIT Compiler
- 정적 컴파일과 동적 컴파일
  - 정적 컴파일은 실행 시점 전에 기계어로 변환하는 컴파일러
  - 동적 컴파일은 실행 중 기계어로 변환하는 컴파일러
  - 정적 컴파일러는 실행 시간이 너무 오래 소요된다는 단점, 동적 컴파일은 실행 도중에 리소스를 사용하여 성능이 떨어진다는 단점이 있다.
  - JVM은 컴파일 타임에는 정적 컴파일러를 사용하고, 런타임에는 단순한 동적 컴파일러가 아닌 더욱 효율적으로 동작하는 JIT Compiler 를 사용
- JIT Compiler의 성능 최적화
  - JIT compiler는 기본적으로 실행 중에 컴파일(동적 컴파일)을 하지만 기계어로 변환된 코드를 캐시에 저장하여 재사용 시 바로 사용하여 좋은 성능을 낸다.
  - 캐시 공간은 작기 때문에 내부에서 자주 수행되는 코드들을 선별하여 캐시 공간에 둔다.

#### GraalVM
- Spring Boot 3.x 버전 부터 AOT 플러그인 제공과 Native 지원에 관한 부분이 확대된다.
- Oracle이 만든 JVM과 JDK로 애플리케이션 성능과 효율성 향상을 제공하는 고성능 런타임 제공
- GraalVM native image는 새로운 Java application 실행 및 배포 방식으로 기존 JVM과 비교했을 때 더 적은 memory와 빠른 실행속도를 가지고 있다.
- 더 빠르고 유지보수가 쉬운 컴파일러 작성, JVM 실행 언어의 성능 향상, 빌드 시간 단축이 목표
- GraalVM 특징
  - AOT 네이티브 이미지 컴파일러(워밍업 시간 단축 - 서버리스 환경에 적합)
    - 소스코드를 실행 전 컴파일하는 방식으로 실행 전에 바이트코드를 기계어로 바꾸는 작업 수행
    - JIT Compiler와 달리 실행 전에 기계어로 변환하는 작업을 수행하기 때문에 실행 시간, 실행 도 중 리소스를 적게 사용할 수 있다는 장점이 있다.
  - Polyglot
    - Python, Ruby, R 등 다양한 언어를 실행할 수 있게 해줄 수 있다.
- GraalVM 의 JVM과 주요 차이점
  - 빌드되는 시간에 정적 분석이 실행
  - lazy classloading 없이 실행 시점에 실행가능하도록 메모리에 올라와 있다

#### 출처
- JIT Compiler(https://kotlinworld.com/307)
- https://docs.spring.io/spring-boot/docs/3.0.0/reference/html/native-image.html#native-image
