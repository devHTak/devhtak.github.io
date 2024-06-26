#### Spring Data R2DBC

#### R2DBC란

- Reactive Relational Databace Connectivity의 약자로 관계형 DB에 대해 reactive API 를 제공
  - JDBC와 달리 적은 스레드와 리소스를 가지고 동시성을 다룰 수 있는 non-blocking을 사용하기 위해 만들어졌다
- R2DBC를 사용하기 위해서는 Project Reactor에 대한 의존성이 필요하다.
  - Mono, Flux 와 같은 응답 객체를 사용한다.
- R2DBC는 interface를 제공하는 Specification 이기 때문에 데이터베이스 벤더 측에서 해당 데이터베이스에 맞도록 구현해야 한다.
- 요구사항
  - JDK 17 이상
  - Spring Framework 6.0.10 이상
  - REDBC

#### Spring Data Repository
- Repository 추상화
  - Repository interface를 추상화 하므로써 boilerplate code(비슷한 코드의 반복 재사용)을 줄여준다
  - CrudRepository: 기본적인 CRUD 기능 제공
  - PagingAndSorgingRepository: 페이징 처리(Page, Pageable) 및 정렬(Sort) 기능 제공
  - ReactiveCrudRepository, ReactiveSortingRepository: Reactive 기능 제공
- Query 메소드 사용
  - findByName 과 키워드를 나열하면 스스로 Query를 생성한다.

#### R2DBC Configuration
- R2dbcAutoConfiguration
  ```java
  @AutoConfiguration(before = { DataSourceAutoConfiguration.class, SqlInitializationAutoConfiguration.class })
  @ConditionalOnClass(ConnectionFactory.class)
  @ConditionalOnResource(resources = "classpath:META-INF/services/io.r2dbc.spi.ConnectionFactoryProvider")
  @EnableConfigurationProperties(R2dbcProperties.class)
  @Import({ ConnectionFactoryConfigurations.PoolConfiguration.class,
  		ConnectionFactoryConfigurations.GenericConfiguration.class, ConnectionFactoryDependentConfiguration.class })
  public class R2dbcAutoConfiguration {
  
  	@Bean
  	@ConditionalOnMissingBean(R2dbcConnectionDetails.class)
  	@ConditionalOnProperty("spring.r2dbc.url")
  	PropertiesR2dbcConnectionDetails propertiesR2dbcConnectionDetails(R2dbcProperties properties) {
  		return new PropertiesR2dbcConnectionDetails(properties);
  	}
  }
  ```
  - Import 부분을 보면 R2DBC에 대한 클래스들을 빈으로 생성해준다.
    - ConnectionFactoryDependentConfiguration 을 보면,
    - 데이터베이스를 연결하는 ConnectionFactory
    - 이를 사용하여 reactive client 인 DatabaseClient를 빈으로 등록해준다.
      ```java
      @Configuration(proxyBeanMethods = false)
      @ConditionalOnClass(DatabaseClient.class)
      @ConditionalOnSingleCandidate(ConnectionFactory.class)
      class ConnectionFactoryDependentConfiguration {
      	@Bean
      	@ConditionalOnMissingBean
      	DatabaseClient r2dbcDatabaseClient(ConnectionFactory connectionFactory) {
      		return DatabaseClient.builder().connectionFactory(connectionFactory).build();
      	}
      }
      ```
      - DatabaseClient 사용방법        
  - 설정 정보(spring.r2dbc.url)
    - 추가 설정에 대한 정보는 R2dbcProperties.class 참고 가능
      
#### 간단한 CRUD 작업
- 의존성
  - Spring Webflux
  - H2 DB
  - spring data r2dbc
- application.yml
  ```yaml
  spring:
    r2dbc:
      url: r2dbc:pool:h2:mem://test
      username: sa
      password:
  logging:
    level:
      org.springframework.r2dbc.core: debug
  ```
- 간단한 CRUD
  - Entity
    ```java
    @Table
    @Getter
    @Setter
    @Builder
    public class Member {
        @Id
        private Long id;
        private String name;
        private LocalDateTime createdAt;
    }
    ```
  - Service
    ```java
    @Service
    @RequiredArgsConstructor
    public class ItemService {
        private final ItemRepository itemRepository;
        public Flux<ItemResponse> getAll() {
            return itemRepository.findAll()
                    .map(ItemResponse::from);
        }
        public Mono<ItemResponse> getById(Long id) {
            return itemRepository.findById(id)
                    .map(ItemResponse::from);
        }
        public Mono<Item> save(String itemName) {
            Item item = Item.builder()
                    .name(itemName)
                    .createdAt(LocalDateTime.now())
                    .build();
            return itemRepository.save(item);
        }
        public Flux<ItemResponse> findByName(String name) {
            return itemRepository.findByName(name)
                    .map(ItemResponse::from);
        }
    }
    ```
  - Repository
    ```java
    public interface ItemRepository extends ReactiveCrudRepository<Item, Long> {
        // name 으로 쿼리되는지 확인
        Flux<Item> findByName(String name);
    }
    ```

#### 연관관계
- 연관관계 지원
    ```
    Spring Data R2DBC aims at being conceptually easy. In order to achieve this, it does NOT offer caching, lazy loading, write-behind, or many other features of ORM frameworks. This makes Spring Data R2DBC a simple, limited, opinionated object mapper.
    ```
  - R2DBC는 Relational Mapping을 지원하지 않는다.
    - JPA에 외래키(Foreign key)이므로 @OneToOne, @OneToMany @ManyToOne 같은 annotation을 사용하지만, R2DBC는 연관관계 매핑을 지원하지 않는다.
    - 그러므로 Lazy loading, Method name을 통한 Join 등이 불가능하다.
  - @Query를 직접 작성하여 Join을 해주어야 한다.
    - DatabaseClient 사용
      ```java
      public Flux<OrderInfoDto> findByMemberId(Member member) {
          var sql = """
                SELECT O.ID AS ORDER_ID
                     , O.ITEM_ID
                     , O.MEMBER_ID
                     , M.NAME AS MEMBER_NAME
                     , I.NAME AS ITEM_NAME
                FROM   ORDERS O
                INNER JOIN MEMBER M
                    ON O.MEMBER_ID = M.ID
                INNER JOIN ITEM I
                    ON O.ITEM_ID = I.ID
                WHERE M.ID = :memberId
                """;
          Flux<OrderInfoDto> result = databaseClient
                .sql(sql)
                .bind("memberId", member.getId())
                .fetch().all()
                .map(row -> new OrderInfoDto(
                        (Long) row.get("ORDER_ID"),
                        (Long) row.get("ITEM_ID"),
                        (Long) row.get("MEMBER_ID"),
                        (String) row.get("MEMBER_NAME"),
                        (String) row.get("ITEM_NAME")
                ));

          return result;
      }
      ```
    - Repository @Query 사용
      ```java
      @Query("""
            SELECT o.id
                , m.id AD memberId
                , o.item_id as itemId
            FROM ord o,
            INNER JOIN MEMBER M
                ON o.member_id = m.id
            """)
      Flux<Orders> findOrderJoinMember();
      ```
- 다른 방법은 없을까?!
  - Converter를 사용하여 매핑해주기
    - WritingConverter: Java Entity class to Database row
    - ReadingConverter: Database row to Java Entity class
  - @Transient
    - Table Column에서 제외되어 Table에 반영되지 않고 Application 에서만 사용할 수있는 필드이다.
    - Relational Mapping이 불가하므로 Transient Field로 관리
  - DatabaseClient 사용
- [소스 보기](https://github.com/devHTak/spring-boot3-study/tree/main/spring-r2dbc)
#### 출처
- https://docs.spring.io/spring-data/r2dbc/docs/current/reference/html/
- https://binux.tistory.com/155
- https://binux.tistory.com/156
