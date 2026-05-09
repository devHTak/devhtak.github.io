## 웸 애플리케이션과 영속성 관리

### 트랜잭션 범위의 영속성 컨텍스트

#### 스프링 컨테이너의 기본 전략
- 스프링 컨테이너는 트랜잭션 범위의 영속성 컨텍스트 전략을 기본으로 사용
  - 트랜잭션 범위와 영속성 컨텍스트 생존 범위가 같다
- 보통 비즈니스 로직을 시작하는 서브스 계층에서 트랜잭션을 시작한다.
  - @Transactional 애노테이션으로 메소드가 시작하기 전 AOP를 통해 트랜잭션이 시작
  - 서비스 종료(메서드 끝) 시 트랜잭션은 커밋 후 flush 호출
  - 예외 발생 시 트랜잭션은 롤백 후 종료하게 될 때 flush를 호출하지 않는다.
- 트랜잭션이 같으면 같은 영속성 컨텍스트를 사용한다.
  - 같은 트랜잭션 범위에서는 다양한 위치에서 엔티티 메니저를 주입받아 사용해도 항상 같은 영속성 컨텍스트를 사용한다.
- 트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다.
  - 같은 엔티티 매니저를 사용해도 트랜잭션에 따라 접근하는 영속성 컨텍스트가 다르다.
  - 스레드마다 각각 다른 트랜잭션을 할당하는 데, 이 때 같은 엔티티 메니저를 호출해도 영속성 컨텍스트가 다르므로 멀티 스레드 상황에도 안전하다.

#### 준영속 상태와 지연 로딩
- 조회한 엔티티가 서비스나 레포지토리에서 트랜잭션에 묶여 있기 떄문에 영속상태이지만, 컨트롤러나 뷰같은 프레젠테이션 계층에서는 준영속 상태가 된다.
- 준영속 상태와 변경 감지
  - 변경 감지 기능은 준영속 상태에서 동작하지 않는다.
- 준영속 상태와 지연 로딩
  - 준영속 상태에서는 지연 로딩 기능이 동작하지 않는다.
  - 아직 초기화되지 않은 프록시 객체를 프레젠테이션 계층에서 호출하여도 실제 데이터를 불러오지 않는다.
  - 해결 방법
    - 뷰가 필요한 엔티티를 미리 로딩
    - OSIV를 사용해 엔티티를 영속 상태로 유지
- 엔티티를 미리 로딩하는 방법
  - Global Fetch
    - FetchType.EAGER로 두는 형태. 불필요한 데이터를 항상 조회하며 N+1 문제가 발생할 수 있다.
    - N+1 문제: Member, Team이 1:N 관계일 때, JPQL로 @Query("select t from Team t")를 하면 팀 조회후 연계된 멤버를 모두 조회하여 N+1 문제 발생
  - JPQL Fetch Join
    - FetchType.LAZY로 두고 JPQL Fetch Join 사용: @Query("select t from Team t join fetch t.member")
    - 화면 별로 조회 메서드가 늘어날 수 있다.
  - 강제 초기화
    - 서비스 계층에서 일부로 객체를 접근하여 초기화한다: member.getTeam().getName()
    - 하이버네이트에 initialize 메소드를 호출하면 프록시를 초기화 할 수 있다. org.hibernate.Hibernate.initalize(member.getTeam());
  - Facade 계층 추가
    - Controller와 Service 계층 사이에 퍼사드 계층을 도입하여 논리적인 의존성을 분리할 수 있다.
    - 퍼사드 계층의 역할과 특징
      - 프레젠테이션 계층과 도메인 모델 계층 간의 논리적 의존성 분리
      - 프레젠테이션 계층에 필요한 프록시 객체 초기화
      - 서비스 계층을 호출해 비즈니스 로직 실행
      - 레포지토리를 직접 호출하여 뷰가 요구하는 엔티티를 찾는다.

#### OSIV (Open Session In View)
- 과거 OSIV: 요청 당 트랜잭션
  - OSIV 핵심은 뷰에서도 지연로딩이 가능하도록 하는 것으로 Filter나 Interceptor에서 트랜잭션을 시작하고 요청이 끝날때까지 트랜잭션을 종료한다.
  - 가장 큰 문제점은 컨트롤러나 뷰 같은 프레젠테이션 계층이 엔티티를 변경할 수 있다는 점이다.
- 스프링 OSIV: 비즈니스 계층 트랜잭션
  - 라이브러리 
    - Hibernate OSIV Servlet Filter: org.springframework.orm.hibernate4.support.OpenSessionInViewFilter
    - Hibernate OSIV Spring Interceptor: org.springframework.orm.hibernate4.support.OpenSessionInViewInterceptor
    - JPA OSIV Servlet Filter: org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter
    - JPA OSIV Spring Interceptor: org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor
  - 동작 원리 (트랜잭션이 사용되지 않을 때, FlushMode.MANUAL(직접 호출) 형태로 하여 flush 하지 않도록 하며 직접 호출해도 트랜잭션 범위 밖에서는 예외(TransactionRequiredException)가 발생한다.)
    - 요청이 들어오면 영속성 컨텍스트를 생성하며 이 때 트랜잭션은 시작하지 않는다.
    - 서비스 계층에서 트랜잭션을 시작하면 이 때 영속성 컨텍스트에 트랜잭션 시작
    - 서비스 계층이 끝나면 트랜잭션을 커밋하여 영속성 컨텍스트를 플러시한다. 이때 영속성 컨텍스트는 살려둔다.
    - 이후 클라이언트의 요청이 끝날 때 영속성 컨텍스트를 종료한다.
  - 장점
    - 트랜잭션이 없어도 읽기 동작은 하기 떄문에 지연로딩 가능
    - 트랜잭션이 없으면 update 처리가 되지 않기 때문에 변경 감지 등이 발생하지 않는다.
  - 주의사항
    - 프레젠테이션 계층에서 엔티티 수정 후 비즈니스 계층에서 트랜잭션 로직 실행하는 경우: 프레젠테이션 계층에서 수정한 내용이 반영되기 떄문에 주의해야 한다.
