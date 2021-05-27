---
layout: post
title: 마스터 스프링 클라우드 - 05. 분산 컨피규레이션
summary: Spring Cloud
author: devhtak
date: '2021-05-27 22:41:00 +0900'
category: Spring Cloud
---

#### 들어가기

- 분산 컨피규레이션에 필요성
  - JAR 파일안에 설정 파일 구성
    - 수정이 필요하면 다시 배포해야 하는 단점이 있다.
  - 외부의 파일 시스템에 저장된 명시적 컨피규레이션 사용
    - spring.config.location 속성으로 구성
    - 다시 배포할 필요는 없지만, 마이크로 서비스가 많을 경우 파일 시스템에 저장된 명시적인 파일을 기반으로 컨피규레이션을 관리하기 어렵다
  
- 분산 컨피규레이션 사용
  - 클라우드 네이티브 환경에서 많이 사용한다.
  - 해당 솔루션을 사용하면 전체 환경에 걸쳐 애플리케이션을 위한 외부 속성을 관리하는 중앙의 단일 장소가 생긴다.
  - 서버는 HTTP와 자원 기반 API 인터페이스를 노출하며 반환된 속성 값에 대해 복호화와 암호화를 수행
  - 클라이언트는 서버로부터 컨피규레이션 설정을 가져온 후 복호화 한다.

- 컨피규레이션 저장소
  - Git(EnvironmentRepository의 기본 구현)
  - SVN
  - 파일 시스템, 볼트 등에서도 사용이 가능하다.

- 다룰 주제
  - 스프링 클라우드 컨피그 서버에 의해 노출되는 HTTP API
  - 서버 측의 다른 타입의 저장소 백엔드
  - 서비스 디스커버리와 통합
  - 스프링 클라우드 버스와 메시지 브로커를 사용하여 컨피규레이션을 자동으로 로딩

#### HTTP API 자원의 소개

- 컨피그 서버는 다양한 방법으로 호출할 수 있는 HTTP API를 제공

  |종단점|설명|
  |---|---|
  |/{application}/{profile}\[/{label}]|JSON 형태로 데이터를 반환, label 입력값은 선택이다.|
  |/{application}-{profile}.yml|YAML 형태를 반환|
  |/{label}/{application-profile}.yml|위 종단점의 변형으로 label 입력값을 선택적으로 입력할 수 있다.|
  |/{application}-{profile}.properties|프로퍼티 파일에서 사용하는 간단한 키/값 형태로 반환|
  |/{label}/{application}-{profile}.properties|위 종단점의 변형으로 label 입력값을 선택적으로 입력할 수 있다.|
  
- 클라이언트의 관점에서 application 입력값은 애플리케이션의 이름으로 spring.application.name 또는 spring.config.name 속성에서 가져온다.
- profile은 활성화된 프로파일 또는 콤마로 분리된 활성화된 프로파일의 목록이다.
- label은 선택적 속성인데, 깃에 경우 브랜치 이름으로 설정한다. 기본 값은 master이다.

- 네이티브 프로파일 지원
  - 파일 시스템 백엔드를 기반으로 시작
    - spring.profiles.active=native로 서버 시작
  - classpath:/, classpath:/config, file:./, file:./config에서 컨피규레이션 파일을 찾는다.
  - 해당 속성 또는 YAML 파일이 JAR 파일 안에 위치할 수 있다는 뜻이다.
  - 테스트를 위해 src/main/resources 아래 config 폴더를 생성했다.
  
  

#### 출처

- 마스터링 스프링 클라우드 

