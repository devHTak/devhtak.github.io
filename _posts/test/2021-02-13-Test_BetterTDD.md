---
layout: post
title: TDD 좀 더 잘하기
summary: 테스트주도개발 TDD실천법과 도구
author: devhtak
date: '2021-02-13 22:41:00 +0900'
category: Test
---

#### 체크 리스트

- 테스트 클래스의 위치
- 테스트 메소드 작성 방식
- 테스트 케이스 접근 방식
- TDD의 한계

#### 테스트 클래스의 위치

- 테스트 대상 소스와 테스트 클래스를 같은 곳에
  - src/bank/Account.java + src/bank/AccountTest.java
  - 테스트 클래스와 같은 패키지에 놓는 가장 기본적인 형태
  - 간단한 테스트를 만들 때 외에는 잘 사용하지 않는다.
  
  - 장점
    - 클래스 이름 규칙을 잘 정했을 경우 테스트 클래스를 찾기 쉽다.
  - 단점
    - 패키지 내에 제품 코드 클래스와 테스트 클래스가 함께 존재하기 때문에 혼란을 줄 수 있다. 배포 시에 테스트 클래스만 발췌해야 하는 불편함이 있다.
  - 선호도
    - 낮음
  
- 테스트 클래스는 하위 패키지로
  - src/bank/Account.java + src/bank/test/AccountTest.java
  - 대상 소스의 하위에 테스트 패키지를 만든는 방법
  
  - 장점
    - 테스트 코드가 가까운 곳에 위치한다.
  - 단점
    - 이미 하위 패키지가 존재할 경우에는 혼란스러울 수 있다. 테스트 클래스와 제품 클래스를 따로 분리하여 배포하는 데 어려움이 있다.
  - 선호도
    - 낮음, 지금은 거의 사용하지 않는다.
  
- 최상위 패키지를 분리
  - /src/main/Account.java + /src/test/AccountTest.java
  - 최상위 패키지부터 업무 코드와 테스트 코드를 분리해서 작성할 수 있게 만들어 준다.
  
  - 장점
    - 업무 코드와 테스트 코드가 섞일 염려가 없다. 테스트 코드만 분리해내기 쉽다.
  - 단점
    - default나 protected로 선언된 메소드들에 대해 테스트 코드를 작성할 수 없다. 어쨌든 배포 시나 패키징시에 test 패키지 이하를 따로 발췌해내야 한다.
  - 선호도
    - 보통
  
- 테스트를 프로젝트로 분리
  - Bank Buisiness Project + Bank Buisiness Test Project
  - 외부 라이브러리 의존 관계를 좀 더 확실하게 구분하고자 프로젝트 단위로 분리
  - 컴포넌트식 개발과 테스트에 흔히 쓰이는 방식
  
  - 장점
    - 제품 코드와 테스트 케이스 코드의 외부 라이브러리 의존 관계를 명확히 분리해낸다. 
    - 제품 코드와 테스트 코드를 프로젝트 레벨로 개별적 배포가 용의
  - 단점
    - 환경 구성 방법이 IDE에 의존적이 된다. Ant 스크립트 등의 외부 툴을 함께 쓸 경우, 클래스패스 관련해서 스크립트 작성이 번거롭다.
  - 선호도
    - 프로젝트 라이브러리에 대해 상세화된 전권행사(Detailed Full control)을 원하는 개발자에게 추천
  
- Maven 스타일
  - 패키지 구조
    - src/main/java + src/main/resources + src/test/java + src/test/resources
  - 실제 폴더 구조에는 클래스 파일 생성 경로
    - target/classes + target/test-classes
  
  - src/main/java
    - 제품 코드가 들어가는 위치
  - src/main/resources
    - 제품 코드에서 사용하는 각종 파일, XML 등의 리소스 파일들
  - src/test/java
    - 테스트 코드가 들어가는 위치
  - src/test/resources
    - 테스트 코드에서 사용하는 각종 파일, XML 등의 리소스 파일들
    
  - 장점
    - 제품 코드에 필요한 리소스와 테스트에 사용하는 리소스를 분리해서 관리하기에 편하다.
    - 메이븐 방식으로 구성된 프로젝트를 접할 때 어색하지 않다.
  - 단점
    - 이런 형태의 구조를 처음 접할 경우 적응하는 데 시간이 걸린다.
  - 선호도
    - 보통
    - Maven 사용자에게는 회피할 수 없는 구조
    
#### 테스트 메소드 작성 방식

- 테스트 대상 메소드와 이름을 1:!로 일치
  - 테스트 대상 코드
    ```java
    public int getBalance() {...} 
    ```
  - 테스트 코드
    ```java
    @Test
    public void testGetBalance() {...}
    ```
  - 장점
    - 테스트 메소드의 숫자가 적어져서 보기 편하고, 대상이 되는 클래스의 메소드와 1:1로 연관지어 생각할 수 있다.
  - 단점
    - 추가적인 테스트 케이스가 하나의 테스트 메소드 내에 전부 존재하게 만들 경우, 메소드 내의 초반 테스트 단정문이 실패하면, 뒤쪽 테스트 케이스들은 실행되지 않는다.
    - 이럴 경우 성공하는 케이스와 실패하는 중간중간 섞여 있을 수 있지만 구별해낼 수 없다.
  
- 테스트 대상 메소드의 이름 뒤에 추가적인 정보 기재
  - 테스트 대상 코드
    ```java
    public int withdraw() {...} 
    ```
  - 테스트 코드
    ```java
    @Test
    public void testWithdraw_마이너스통장인출() {...}
    @Test
    public void testWithdraw_잔고가0원일때() {...}
    ```
  - 장점
    - 더 다양한 케이스별로 성공/실패를 알 수 있다.
    - 케이스마다 독립적으로 수행할 수 있어 오류의 가능성이 좀 더 줄어든다.
  - 단점
    - 테스트 케이스만큼 메소드를 만들기 때문에 메소드 숫자가 많아진다.
    - 테스트 메소드 이름을 짓는데 상당한 요령이 필요하다.
    - 케이스마다 리소스 초기화(setup) 작업이 필요하다.

- 테스트 시나리오에 집중
  - 테스트 대상 코드
    - 특정한 메소드를 대상으로 하기보다는 테스트 시나리오가 대상이 된다.
  - 테스트 코드
    ```java
    @Test
    public void vip고객이_인출할때_수수료계산() {...}
    @Test
    public void 일반고객이_인출할때_수수료계산() {...}
    ```
  - 통합 테스트나 사용자 테스트에 가까운 형태로 테스트 케이스를 만들 필요가 있을 때 사용
  - 작성 대상 클래스의 메소드 구현을 위해 사용한다기보다는 테스트 시나리오에 집중해서 대상 클래스 자체의 통합적인 기능 테스트
  - 특성상 일반적으로 선조건 -> 수행 -> 예상 결과 식의 시나리오를 갖는다
  
  - 장점
    - 테스트 케이스를 시나리오에 따라 체계적으로 작성할 수 있다.
    - 테스트 클래스를 하나의 업무단위 테스트 단위처럼 문서화할 수 잇다.
  - 단점
    - 테스트 대상 클래스의 단위 메소드 구현 시에 사용하기에는 다소 무리가 따른다.

#### 테스트 케이스 작성 접근 방식

- 무엇을 테스트 케이스로 작성할 것인가?
  - 해피데이 시나리오
    - 정상적인 흐름일 때 동작해야 하는 결과값을 선정해 놓는 것
  - 블루데이 시나리오
    - 발생할 수 있는 예외나 에러 상황에 대한 결과값을 적어 사용
  - 다음과 같은 질문을 해보자
    - 결과가 옳은가?
    - 모든 경계조건이 옳은가?
    - 역(inverse) 관계를 확인할 수 있는가?
    - 다른 수단을 사용해서 결과를 교차확인할 수 있는가?
    - 에러 조건을 강제로 만들어 낼 수 있는가?
    - 성능이 한도 내에 있는가?

#### TDD의 한계

- 동시성 문제
  - 동시성이 걸려 있는 코드에 대한 테스트 케이스 작성은 테스트 자체를 무결하게 유지하기 어렵다.
  
- 접근제한자(private/ protected 메소드)
  - 테스트 클래스에서 private 메소드에 대한 접근이 안되는 경우
  - 현재는 'public 메소드만 테스트해도 무방하다'는 경향이 있다. 그 이유는 public 메서드에서 해당 private 메서드를 호출해서 사용하기 때문이다.
 
** 참고 서적: 테스트주도개발 TDD실천법과 도구
