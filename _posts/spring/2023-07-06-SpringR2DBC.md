#### Spring Data R2DBC

#### R2DBC란

- Reactive Relational Databace Connectivity의 약자로 관계형 DB에 대해 reactive API 를 제공
  - JDBC와 달리 적은 스레드와 리소스를 가지고 동시성을 다룰 수 있는 non-blocking을 사용하기 위해 만들어졌다
- R2DBC를 사용하기 위해서는 Project Reactor에 대한 의존성이 필요하다.
  - Mono, Flux 와 같은 응답 객체를 사용한다.
- 요구사항
  - JDK 17 이상
  - Spring Framework 6.0.10 이상
  - REDBC

#### Spring Data Repository
- Repository 추상화
  - Repository interface를 추상화 하므로써 boilerplate code(비슷한 코드의 반복 재사용)을 줄여준다
  - CrudRepository: 기본적인 CRUD 기능 제공
  - PagingAndSorgingRepository: 페이징 처리(Page, Pageable) 및 정렬(Sort) 기능 제공
  - Reactive*, RxJava3*: Reactive 기능 제공
- Query 메소드 사용
  - findByName 과 키워드를 나열하면 스스로 Query를 생성한다.
- Repository 생성

#### 출처
- https://docs.spring.io/spring-data/r2dbc/docs/current/reference/html/
