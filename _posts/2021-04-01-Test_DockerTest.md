---
layout: post
title: 더 자바 테스트3. 도커와 테스트
summary: Testcontainers
author: devhtak
date: '2021-04-01 21:41:00 +0900'
category: Test
---

#### 테스트 환경 구축

- Mock 객체에서 실제 객체를 사용하도록 테스트 코드를 변경하였다.
  ```java
  @SpringBootTest
  @ExtendWith(MockitoExtension.class)
  @ActiveProfiles("test")
  public class StudyServiceTest {
      @Mock MemberService memberService;
      @Autowired StudyRepository studyRepository;
      
      // ...
  }
  ```
  - @SpringBootTest 는 @ExtendWith(SpringExtension.class) 확장을 사용하기 때문에 @SpringBootApplication 이하에 빈을 사용할 수 있도록 한다.
  - 이 중, StudyRepository를 사용하기 위해서는 DB에 접속해야 한다.

- application.properties
  ```
  spring.jpa.hibernate.ddl-auto=update
  
  spring.datasource.url=jdbc:postgresql://localhost:5432/study
  spring.datasource.username=study
  spring.datasource.password=study
  ```
  
  - Docker를 통해 postgresql을 띄어야 접속이 가능하다.
  
- 테스트에서 H2 DB에서 postgresql 사용하기
  - DB마다 @Transactional에 propagation, isolation 등의 기능이 약간 다르다.
  - 테스트 환경에서는 운영환경과 똑같은 DB를 사용해야 해당 차이로 오는 버그를 사전에 찾고 해결할 수 있다.
  - test/resources/application-test.properties
    ```
    spring.datasource.url=jdbc:postgresql://localhost:15432/study-test
    spring.datasource.username=studytest
    spring.datasource.password=studytest
    
    spring.jpa.hibernate.ddl-auto=create-drop
    ```

- 이렇게 설정하기 위해서는 테스트할 때마다 postgresql study-test 컨테이너를 띄어주고, 중단하는 등의 사전작업이 필요하다.
- 해당 사전작업을 해결해주는 오픈소스가 Testcontainers 이다.

#### Testcontainers 소개

- 테스트에서 도커 컨테이너를 실행할 수 있는 라이브러리
  - https://www.testcontainers.org/
  - 테스트 실행 시 DB를 설정하거나 별도의 프로그램 또는 스크립트를 실행할 필요가 없다.
  - 보다 Production에 가까운 테스트를 만들 수 있다.
  - 단점은 테스트가 느려진다.
  
#### Testcontainers 설치

- Testcontainers JUnit5 지원 모듈 의존성 추가
  ```
  <dependency>
      <groupId>org.testcontainers</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>1.15.1</version>
      <scope>test</scope>
  </dependency>
  ```

  - @Testcontainers
    - JUnit5 확장팩으로 테스트 클래스의 @Container를 사용한 필드를 찾아서 컨테이너 라이프사이클 관련 메소드를 실행해준다.
  - @Container
    - 인스턴스 필드에 사용하면 모든 테스트마다 컨테이너를 재시작하고, 스태틱 필드에 사용하면 클래스 내부 모든 테스트에서 동일한 컨테이너를 재사용한다.
    - 여러 모듈을 제공하는 데, 각 모듈은 별도로 설치해야 한다.
      - PostgreSQL 설치: https://www.testcontainers.org/modules/databases/
        ```
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <version>1.15.1</version>
            <scope>test</scope>
        </dependency>
        ```
- PostgreSQLContainer 실행

  ```java
  @SpringBootTest
  @ExtendWith(MockitoExtension.class)
  @ActiveProfiles("test")
  @TestContainers
  public class StudyServiceTest {
      @Mock MemberService memberService;
      @Autowired StudyRepository studyRepository;

      static PostgreSQLContainer postgreSQLContainer = new PostgreSQLContainer()
          .withDatabaseName("studytest");

      @BeforeAll
      static void beforeAll() {
          postgreSQLContainer.start();        
      }

      @AfterAll
      static void afterAll() {
          postgreSQLContainer.stop();
      }
      
      @BeforeEach
      void beforeEach() {
          studyRepository.deleteAll();
      }
      // ...
  }
  ```
  - static 필드로 선언하면 각각 테스트마다 컨테이너는 한번만 실행된다.
    - static이 없으면 테스트마다 컨테이너가 삭제되고 생성되기 때문에 오래걸린다.
  - 컨테이너를 생성하는 데 오래걸리기 때문에 성능 상 static 을 사용
    - 대신, beforeEach를 통해 DB 데이터 삭제 필요

- 지정한 위치에 컨테이너 생성하기
  - test/resources/application-test.properties
    ```
    spring.datasource.url=jdbc:tc:postgresql://studytest
    spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver
    ```
    
#### Testcontainers 기능 살펴보기

- 컨테이너 만들기
  ```java
  GenericContainer container = new GenericContainer(String imageName);
  ```
- 네트워크
  - 호스트 포트 설정
    ```java
    withExposedPorts(int...)
    ```
  - 컨테이너 내부 매핑된 포트 확인
    ```java
    getMappedPort(int)
    ```

- 환경 변수 설정
  ```java
  withEnv(key, value)
  ```
  
- 명령어 실행
  ```java
  withCommand(String cmd...)
  ```

- 예제
  ```java
  @Container
  static GenericContainer postgreSQLContainer = new GenericContainer("postgres")
      .withExposePort(15432)
      .withEnv("POSTGRES_DB", "studytest");
  ```

- 사용할 준비가 됐는지 확인
  ```java
  waitFor(Wait)
  ```
  - Wait.forHttp(String URL): Convenience method to return a WaitStrategy for an HTTP endpoint.
  - Wait.forLogMessage(String reges, int times): Convenience method to return a WaitStrategy for log messages.

- 로그 살펴보기
  - getLogs(): 현재 컨테이너의 로그를 가져오는 것
  - followOutput(): stream으로 로그를 가져오는 것    
  - 로그 사용하기
    - 롬복: @Slf4j
    - Logger LOGGER = LoggerFactory.getLogger(StudyServiceTest.class);
  
  ```java
  @BeforeAll
  static void beforeAll() {
      Slf4jLogConsumer logConsumer = new Slf4jLogConsumer(log);
      postgreSQLContainer.followOutput(logConsumer);
  }

  @BeforeEach
  void beforeEach() {
      System.out.println(postgreSQLContainer.getLogs());
  }
  ```
  
#### 컨테이너 정보를 스프링 테스트에서 참조하기

- @ContextConfiguration
  - 스프링이 제공하는 애노테이션으로 스프링 테스트 컨텍스트가 사용할 설정 파일 또는 컨텍스트를 커스터마이징할 수 있는 방법 제공
  
- ApplicationContextInitializer
  - 스프링 ApplicationContext를 프로그래밍으로 초기화할 때 사용할 수 있는 콜백 인터페이스
  - 특정 프로파일을 활성화하거나, 프로퍼티 소스를 추가하는 등의 작업을 할 수 있다.
  
- TestPropertyValues
  - 테스트용 프로퍼티 소스를 정의할 때 사용
  
- Environment
  - 스프링 핵심 API로, 프로퍼티와 프로파일을 담당한다.
  
- 전체 흐름
  - Testcontainer를 사용해서 컨테이너 생성
  - ApplicationContextInitializer를 구현하여 생선된 컨테이너에서 정보를 축출하여 Environment에 넣어준다.
  - @ContextConfiguration을 사용해서 ApplicationContextInitializer 구현체를 등록한다.
  - 테스트 코드에서 Environment, @Value, @ConfigurationProperties 등 다양한 방법으로 해당 프로퍼티를 사용한다.
  
- 예제
  ```java
  @SpringBootTest
  @ExtendWith(MockitoExtension.class)
  @ActiveProfiles("test")
  @TestContainers
  @Slf4j
  @ContextConfiguration(initializers= StudyServiceTest.ContainerPropertyInitializer.class)
  public class StudyServiceTest {
      @Mock MemberService memberService;
      @Autowired StudyRepository studyRepository;
      @Autowired Environment environment; // Spring에 있는 Environment 사용

      static PostgreSQLContainer postgreSQLContainer = new PostgreSQLContainer()
          .withDatabaseName("studytest");

      @BeforeAll
      static void beforeAll() {
          postgreSQLContainer.start();        
      }

      @AfterAll
      static void afterAll() {
          postgreSQLContainer.stop();
      }
      
      @BeforeEach
      void beforeEach() {
          studyRepository.deleteAll();
          System.out.println(environment.getProperty("container.port");
      }
      
      static class ContainerPropertyInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
          @Override                                                                                                       
          public void initialize(ConfigurableApplicationContext applicationContext) {                                     
            // TODO Auto-generated method stub                                                                          
            TestPropertyValues.of("container.port=" + postgreSQLContainer.getMappedPort(5432))                          
              .applyTo(applicationContext.getEnvironment());
          }                                                             
      }
  }
  ```
    
