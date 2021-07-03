---
layout: post
title: Micro Service Architecture
summary: msa 개요
author: devhtak
date: '2021-06-05 14:41:00 +0900'
category: msa
---

#### Cloud Native Architecture

- 확장 가능한 아키텍처
  - 시스템의 수평적 확장에 유연
  - 확장된 서버로 시스템의 부하 분산, 가용성 보장
  - 시스템, 서비스 어플리케이션 단위의 패키지(컨테이너 기반 패키지)
  - 모니터링

- 탄력적 아키텍처
  - 서비스 생성, 통합, 배포
    - 비즈니스 환경 변화에 대응 시간 단축
  - 분활된 서비스 구조
  - 무상태 통신 프로토콜
  - 서비스의 추가와 삭제 자동 감지
  - 변경된 서비스 요청에 따라 사용자 요청 처리(동적 처리)

- 장애격리(Fault Isolation)
  - 특정 서비스에 오류가 발생해도 다른 서비스에 영향을 주지 않음

- Cloud Native Application
  - Microservices로 개발
  
  - CI/CD
    - 지속적 통합(Continous Integration)
      - 통합 서버, 소스관리, 빌드 도구, 테스트 도구 
      - ex) Jenkins, Team CI, Travis CI
    - 지속적 배포(Continuous Delivery, Continuous Deployment)
      - pipeline 구축
      
  - DevOps
    - Development + Operation + Quality Assurance
    - Plan - Create - Verify - Package - Release - Configure - Monitor를 통합함으로써 빠르게 통합, 배포, 테스트할 수 있도록 한다.
    
  - Container

- 12 Factors: Cloud native application을 만들고 서비스할 때 지켜야 할 12가지 요소
  - Base Code
  - Depedency isolation
  - configurations
  - linkable backing services
  - stages of creation (build, release and run)
  - stateless processes
  - port binding
  - concurrency
  - desposability
  - development & production parity (dev/prod parity)
  - logs
  - admin processes from eventual progresses
  ++ 3 factors
    - API First, Telemetry, Authentication and authorization
    
#### MSA 등장 배경

- Monotlith Application
  - 대규모 개발자에 의한 개발/배포/유지보수의 어려움이 존재
  - 복잡성/영향도
    - 초반에 구축이 잘 되었어도 수년이상 운영되면서 점차 스파게티화
    - 통합 빌드/배포로 개발자에게 영향을 미친다.
    - 영향도 파악 및 전체 시스템 구조 파악의 어려움
    - 추가/변경 요구사항에 대한 신속 대응 어려움
  - 배포
    - 업무별 배포가 아닌, 통일된 빌드, 배포 시간 / 주기
  - 확장/장애
    - 특정 업무/시점에 대한 부분적/일시적 확장 어려움
    - App/DB Server 확장의 효율성 저하
    - 각 업무별 특성에 알맞은 확장/정책수립 어렵다.
    - 전체 업무로 장애 전파
  - 기술 적합성
    - 특정 업무에 적합한 기술 표준 변경 또는 신기술 적용 어려움
    - 차세대 개발까지 대기, 업무별 적용 어려움

- Monolithic application -> Internally componentized application -> Microservices application
  - App/DB를 물리적으로 분리
  - 각각의 프로세스로 동작
  
- MSA란?
  ```
  MSA는 독립적인 배포 및 확장이 가능한 단위인 마이크로서비스를 지원하는 아키텍처이며 마이크로서비스를 고객의 MSA 적용 목적에 최적화된 단위로 도출하고 설계하는 것이 핵심 역량이다.
  마이크로 서비스 아키텍처는 하나 또는 소수의 큰 시스템을 작은 크기의 서비스로 나누고, 서비스 별로 개별적인 데이터 저장소를 소유하고 독립된 실행 환경을 구축함으로써 서비스 단위로 독립적인 개발, 빌드, 테스트, 배포, 모니터링 확장이 가능한 아키텍처이다.
  ```
  
- MSA 목적
  - Micronservice architecture <-> Cloud-native platforms <-> DevOps Processes
    - MSA
      - Buisiness Agility(민첩성)
      - Rapid/Frequent/Reliable Software Delivery
    - Cloud-native platforms
      - 자원 효율성 증가
      - auto-scaling
      - 보안, API Gateway, Monitoring 등 부가 기능 활용 가능
    - DevOps
      - 운영하면서 지속적으로 개발, 개발과 운영을 결합한 합성어
      - 개발, 빌드, 테스트 및 운영이 용이하도록 사고방식의 변화, 협업, 향상 및 긴밀한 통합 강조
      - 기업의 조직과 문화 등 기술 외적인 변화 필요
        - 역할 중심의 개발 팀에서 개발 및 운영하는 기능(Service) 단위로 팀을 조정

#### MSA 장점
  - 물리적 분리/독립
    - 업무 상황/특징에 맞게 서비스 추가/변경
    - 논리적 분리의 한계 극복
    - 복잡도 감소
    - 서비스 별 독립적인 개발/배포/운영 가능
  - 배포
    - Rapid/Frequent/Reliable Software Delivery
    - 서비스 별 개별 배포
    - 요구사항 신속/전시 반영
  - 장애
    - 장애 전이 방지, 격리 가능
    - 장애가 개별적으로 발생, 전체 운영 중단으로 확대되지 않음 > 장애로 인한 Buisiness 피해 최소화
  - 확장
    - 업무 확장성, 물리적 확장성(scaling) 용이
    - 특정 업무, 특정 시점을 고려한 확장성 용이
    - 사업환경 변화 대응력 향상
  - 기술/환경/조직
    - 신기술 또는 업무 특성에 맞는 최적 기술 적용 유연

  - 고려사항
    - 서비스별(업무중심) Application/DB의 물리적 분리
    - 각 서비스별 프로세스에서 실행
    - 모듈화, Object, CBD 등의 논리적 구분이 잘 되어있다면 서비스 분리도 유리
    - MSA 특징 및 목적에 맞는 서비스 분리 기준 필요
    - MSA 도입으로 인한 추가 설계, 기반기술, 조직 재정비 등 필요(Not Free)
      - 분리된 서비스 간의 통신방법/비용 
      - 분산 트랜잭션, 분산 데이터 조합 방안
      - 테스트자동화
      - DevOps 조직
    - MSA의 맹목적 도입이나 부적절한 서비스 분리는 Monolith에 비해 더 많은 복잡성, 성능, 신뢰성 등의 문제를수 반할수있으므로 MSA 도입 목적과 특징을 고려한 신중한 판단 필요

#### MSA 단점
  - 성능(서비스 간 호출 시 API 통신)
    - 통신 포맷인 Json / Xml 메시지를 Java Object(VO 등) 데이터 모델로 변환하는 오버헤드 발생
    - 메시지들이 네트워크를 통해 전송되기 때문에 시간이 더 추가소요 (통신비용증가)
      - Latency Time(대기시간, 호출시간)
  - 테스트(서비스 들 각각 분리, 타 서비스에 대한 종속성)
    - 특정 시나리오, 기능 테스트 시 여러 서비스에 걸쳐 테스트를 진행
    - 테스트 환경 구축, 문제 발생 시 여러 시스템을 동시에 점검해야 하는 등 테스팅 복잡도 증가
    - 테스트 자동화 방안 필요
  - 트랜잭션(분산환경으로 DBMS 내 트랜잭션 처리(Commit, Rollback) 이용 불가)
    - API 기반의 여러 서비스를 하나의 트랜잭션으로 묶기 어려움
  - 데이터 조합(Query, 서비스 별 데이터 분리 저장)
    - 여러 서비스에 걸쳐 분산된 데이터를 한번에 조회하기 어려움(별도 방안 필요)
  - 복잡도(분산환경, 트랜잭션, Query 등을 위한 별도의 설계, 개발, 테스트 필요)
    - 서비스 호출 추적, 디버깅(장애추적), 모니터링 등의 어려움
    - 전체적인 복잡도 증가

#### 출처
- Clout Native Architecture 책(톰 랴쥬스키 , 카말 아로라 , 에릭 파, 피윰 조누즈 , 홍성민 저)
- 인프런 강의: Spring Cloud로 개발하는 마이크로 서비스 
