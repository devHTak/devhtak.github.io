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
  - ReactiveCrudRepository, ReactiveSortingRepository: Reactive 기능 제공
- Query 메소드 사용
  - findByName 과 키워드를 나열하면 스스로 Query를 생성한다.
- Repository 생성

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
- application.yml
      
#### 출처
- https://docs.spring.io/spring-data/r2dbc/docs/current/reference/html/
