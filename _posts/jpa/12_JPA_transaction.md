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
   
### 엔티티 비교
- 영속성 컨텍스트 내부에는 엔티티 인스턴스를 보관하기 위한 1차 캐시가 있다. 1차 캐시는 영속성 컨텍스트와 생명주기를 같이 한다.

#### 영속성 컨텍스트가 같을 때 엔티티 비교 
- 아래 모두 비교가 가능하다
  - 동일성(==)
  - 동등성(Object.equals())
  - 데이터베이스 동등성

#### 영속성 컨텍스트가 다를 때 엔티티 비교 
- 동일성 비교에 실패한다.
  - 영속성 켄텍스트가 다른 경우 1차 캐시에서 관리되는 엔티티도 다르기 떄문에 동일성은 다르다
- 동등성, 데이터베이스 동등성은 같다.
  - 관리하는 1차 캐시가 달라 다른 hashCode를 갖지만 실제 Id, 값들은 동일하기 때문에 동등성, 데이터베이스 동등성은 성공한다.
 
#### 프록시 비교
- 영속성 컨텍스트와 프록시
  - 영속성 컨텍스트는 프록시로 조회된 엔티티에 대해 같은 엔티티를 찾는 요청이오면 원본 엔티티가 아닌 처음 조회한 프록시를 반환하여 동일성을 보장한다.
- 프록시 타입 비교
  - 프록시는 원본 엔티티를 상속받아 만들어지므로 조회한 엔티티의 타입을 비교할 때 == 대신 instanceof 를 사용해야 한다.
- 프록시 동등성 비교
  - 엔티티의 동등성을 비교할 때에는 Primary Key에 대한 equals 메서드를 오버라이딩하고 비교하면 된다.
  - 근데, equals를 비교할 때 타입을 비교하는 로직에서 ==를 사용하게 되는 경우 프록시는 동등성 실패가 발생할 수 있다.
    ```java
    // if(this.getClass() != obj.getClass()) return false;
    if(!(this.getClass() instanceof Member)) return false;
    ```

### 트랜잭션과 락

#### 트랜잭션과 격리 수준
- ACID
  - Atomic: 원자성 - 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인것처럼 모두 성공하든가 모두 실패해야 한다
  - Consist: 일관성 - 트랜잭션은 일관성있이 DB 상태를 유지해야 한다.
  - Isolation: 격리성 - 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다.
  - Durability: 영구성 - 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다.
- Isolation Level
  - Read Uncommited
    - Dirty Read: 커밋하지 않은 데이터를 읽을 수 있다.
  - Read Commited
    - 커밋한 데이터를 읽는다.
    - Non Repeatable Read: 아이디가 1인 회원 데이터를 읽고, 다시 읽었을 때 다른 트랜잭션이 회원 정보를 수정했다면 변경된 데이터를 읽는다.
  - Repeatable Read
    - 한번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회된다.
    - Phantom Read: 10살 이하 회원을 조회하였을 때, 다른 트랜잭션이 5살 회원을 insert 했으면 다시 조회했을 때 신규 row가 있다.
  - Serializable
    - 가장 엄격한 트랜잭션 격리 수준, Phantom Read 가 발생하지 않지만 동시성 처리 성능이 급격히 떨어진다.

#### 낙관적 락과 비관적 락
- 낙관적 락
  - 트랜잭션 대부분은 충돌이 발생하지 않는다고 낙관적 락으로 가정하는 방법
  - DB가 제공하는 락을 사용하는 것이 아닌 버전 관리 기능으로 처리하는 것으로 애플리케이션이 제공하는 락이다.
  - 낙관적 락은 트랜잭션을 커밋하기 전까지 트랜잭션 충들을 알 수 없다.
- 비관적 락
  - 트랜잭션의 충돌이 발생한다고 가정하고 락을 걸고 보는 방법 (대표적으로 select for update)
- 두번의 갱신 분실 문제
  - 마지막 커밋만 인정하기: 마지막에 커밋한 사용자의 내용만 인정
  - 최초 커밋만 인정하기: 처음에 커밋한 사용자의 내용을 인정하고 다음 수정에 오류를 발생시킨다.
  - 충돌하는 갱신 내용 병합하기: 수정사항 병합
  - 버전 관리를 하면 최초 커밋만 인정하는 형태로 처리할 수 있다.
- @Version
  ```java
  @Entity
  public class Board {
    @Id private Long id;
    // ...
    @Version private Long version;
  }
  ```
  - Long, Integer, Short, Timestamp 타입에 사용할 수 있다.
  - 트랜잭션1에서 조회(version: 1)하여 수정할 때 트랜잭션2도 조회(version: 1)하여 수정(version: 2)하게 되면 트랜잭션1에서의 수정은 버전이 달라 수정이 불가하다.
    - 버전 정보가 다르면 예외가 발생한다.
  - 버전 정보를 사용하면 최초 커밋만 인정하게 된다.
  - 버전 정보 비교 방법
    - update Board set title = ?, version = ? where id = ? and version = ?
    - 버전은 엔티티의 값을 변경하면 증가한다.
- JPA 락 사용
  - LockModeType
    - OPTIMISTIC
      - 낙관적 락 사용
      - 이 옵션을 사용하면 조회만 해도 버전 정보를 체크한다. 커밋할 때 버전 정보를 확인하여 현재 엔티티의 버전과 같은지 검증, 만약 같지 않으면 예외가 발생한다.
    - OPTIMISTIC_FORCE_INCREMENT
      - 낙관적 락 + 버전 정보를 강제로 증가한다.
      - 논리적인 단위의 엔티티 묶음을 관리할 수 있다. 계시물과 첨부파일이 연관관계를 맺을 때 첨부파일을 추가하면 게시물의 버전도 강제로 증가된다.
    - PESSIMISTIC_READ
      - 비관적 락, 읽기 락을 사용한다.
      - lock in share mode - 데이터를 반복 읽기만 하고 수정하지 않는 용도로 락을 걸 때 사용
    - PESSIMISTIC_WRITE
      - 비관적 락, 쓰기 락을 사용한다.
      - select for update를 사용해 락을 건다.
      - 락이 걸린 로우는 다른 트랜잭션이 수정할 수 없다.
    - PESSIMISTIC_FORCE_INCREMENT
      - 비관적 락, 버전정보를 강제로 증가한다.
      - 비관적 락중에 유일하게 버전 정보를 사용한다. for update 사용
    - NONE
      - 락을 걸지 않는다.
      - @Version 이 있는 경우, 낙관적 락이 적용된다.

### 2차 캐시
- 1차 캐시
  - 영속성 컨텍스트에 있다.
  - 엔티티 메니저로 조회하거나 변경하는 모든 엔티티는 1차 캐시에 저장되며 1차 캐시에 있는 엔티티의 변경 내역을 데이터베이스에 동기화 된다.
- 2차 캐시
  - 애플리케이션이 공유하는 캐시를 JPA는 공유 캐시라 하는 데 일반적으로 2차 캐시라 한다.
- 캐시 모드 설정
  ```java
  @Cacheable // javax.persistence.Cacheable
  @Entity
  public class Member {
    @Id private Long id;
  }
  ```
  - SharedCacheMode 캐시 모드 설정
    ```yaml
    spring:
      jpa:
        properties:
      	  javax:
              persistence:
                sharedCache:
                  mode: ENABLE_SELECTIVE
    ```
    - ALL: 모든 엔티티를 캐시한다.
    - NONE: 캐시를 사용하지 않는다.
    - ENABLE_SELECTIVE: @Cacheable(true)로 설정된 엔티티만 캐시 적용
    - DISABLE_SELECTIVE: 모든 엔티티를 캐싱하는 데, @Cacheable(false)로 설정된 엔티티는 제외한다.
    - UNSPECIFIED: JPA 구현체가 정의한 설정을 따른다.
- CacheRetrieveMode
  - 캐시 조회 모드 설정 옵션
  - USE (캐시 조회), BYPASS(캐시를 무시하고 DB 조회) 가 있다.
- CacheStoreMode
  - 캐시 보관 모드 설정 옵션
  - USE: 조회한 데이터를 캐시에 저장하며 이미 캐시에 있는 경우 최신 정보로 수정하지 않는다
  - BYPASS: 캐시에 저장하지 않는다.
  - REFRESH: 조회한 데이터를 캐시에 저장하며 이미 캐시에 있는 경우 최신 정보로 수정
- 엔티티 캐시와 컬렉션 캐시
  - CacheConcurrencyStrategy 속성 - @Cache(usage=CacheConcurrencyStategy.READ_WRITE) - 캐시 동시성 전략
    - NONE: 캐시 설정 하지 않는다.
    - READ_ONLY: 읽기전용 설정, 등록 삭제만 가능하다.
    - NONSTRICT_READ_WRITE: 엄격하지 않은 읽고 쓰기 전략??
    - READ_WRITE: 읽기, 쓰기가 가능하고 READ COMMITTED 수준의 격리수준을 보장
    - TRANSACTIONAL: 컨테이너 관리 환경에서 사용할 수 있다. 설정에 따라 REPEATABLE_READ 격리수준을 보장 할 수도 있다.
