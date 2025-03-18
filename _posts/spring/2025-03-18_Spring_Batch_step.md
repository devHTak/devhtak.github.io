---
layout: post
title: Spring Batch Step - Chunk, Tasklet
summary: Spring Batch
author: devhtak
date: '2025-03-18 21:41:00 +0900'
category: Spring
---
#### Step의 ChunkModel과 TaskletModel
- Chunk Model은 데이터를 작은 단위로 나눠 반복적으로 처리하는 방식으로 데이터를 읽고(ItemReader) 처리한 후(ItemProcessor), 결과를 저장하거나 파일처리(ItemWriter)한다.
- Tasklet Model은 데이터를 하나씩 처리하거나 한번의 태스크로 배치를 수행할 때 이용되며 한 레코드씩 읽어 써야하는 경우 적합하다.
- Chunk 는 대량의 데이터를 안정적으로 처리할 때 유리하며, Tasklet 은 작은 데이터를 간편하게 처리할 때 유리

##### ChunkModel
- 처리할 데이터를 일정단위로 처리하는 방식으로 ChunkOrientedTasklet은 청크처리를 지원하는 Tasklet의 구체적인 클래스 역할 수행
- 청크에 포함될 데이터의 최대 레코드 수(청크 size)는 본 클래스의 commit-interval이라는 설정값을 이용하여 조정 가능
- ItemReader, ItemProcessor, ItemWriter 구현체를 각각 호출하며 청크 단위에 따라 반복 실행한다
  - 청크 크기만큼 ItemReader에서 읽어온 후 ItemProcessor로 전달 및 처리, ItemWriter에서 저장, 파일 처리 수행(반복)
- ItemReader
  - Spring에서는 다양한 구현체를 제공
  - FlatFileItemReader
    - 플랫파일(구조화되지 않은 파일)로 부터 읽어 들인다.
    - 읽어들인 데이터를 객체로 매핑하기 위해 delimeter를 기준으로 매핑 룰을 이용하여 객체로 매핑
  - StaxEvenItemReader
    - XML 파일을 읽어들인다.
  - JdbcPagingItemReader/JobcCursorItemReader
    - JDBC를 사용해 SQL 실행하고 데이터베이스의 레코드를 읽는다.
    - 데이터베이스에서 많은 양의 데이터를 처리해야 하는 경우에는 메모리에 있는 모든 레코드를 읽는 것을 피하고, 한 번의 처리에 필요한 데이터만 읽고 폐기하는 것이 필요하다.
    - JdbcPagingItemReader는 JdbcTemplate을 이용하여 각 페이지에 대한 SELECT SQL을 나누어 처리하는 방식으로 구현된다.
    - JdbcCursorItemReader는 JDBC 커서를 이용하여 하나의 SELECT SQL을 발행하여 구현된다.
  - MyBatisCursorItemReader/MyBatixPagingItemReader
    - MyBatis를 사용하여 데이터베이스의 레코드를 읽는다.
    - JPA를 이용한 구현체도 제공
      - JpaPagingItemReader, HibernatePagingItemReader, HibernateCursor 제공
  - JmsItemReader/AmqpItemReader
    - JMS, AMQP 에서 읽어온다.
- ItemProcessor
  - Spring에서는 다양한 구현체를 제공
  - PassThroughItemProcessor
    - 아무런 작업도 수행하지 않으며 입력된 데이터의 변경이나 처리가 필요하지 않는 경우 사용
  - ValidatingItemProcessor
    - 입력된 데이터를 체크하며, 규칙을 구현하려면 Validator를 구현해야 한다.
    - 일반적인 Validator의 어댑터인 SpringValidator와 org.springframework.validation 의 규칙 제공
  - CompositeItemProcessor
    - 동일한 입력 데이터에 대해 여러 ItemProcessor를 순차적으로 실행
    - ValidatingItemProcessor를 사용하여 입력 확인을 수행한 후 비즈니스 로직을 실행하려는 경우 활성화
- ItemWriter
  - Spring에서는 다양한 구현체를 제공
  - FlatFileItemWriter
    - 처리된 Java객체를 CSV 파일과 같은 플랫 파일로 작성한다.
    - 파일 라인에 대한 매핑 규칙은 구분 기호 및 개체에서 사용자 정의로 사용할수도 있다.
  - StaxEventItemWriter
    - XML파일로 자바 객체를 쓰기할 수 있다.
  - JdbcBatchItemWriter
    - JDBC를 사용하여 SQL을 수행하고 자바 객체를 데이터베이스에 쓰기한다.
    - 내부적으로 JdbcTemplate를 사용하게 된다.
  - MyBatisBatchItemWriter
    - Mybatis를 사용하여 자바 객체를 데이터베이스로 쓰기한다.
    - MyBatis-Spring 는 MyBatis에 의해서 제공되는 라이브러리를 이용한다.
  - JmsItemWriter / AmqpItemWriter
    - JMS혹은 AMQP로 자바 객체의 메시지를 전송한다.

##### TaskletModel
- chunk model은 큰 데이터를 분할해서 ItemReader --> ItemProcessor --> ItemWriter 순으로 처리가 필요한경우 매우 유용하다.
- 반면 청크 단위의 처리가 딱 맞지 않을경우 Tasklet Model이 유용하다.
  - 한번에 하나의 레코드만 읽어서 쓰기해야하는 경우 Tasklet Model이 적합하다.
  - 사용자는 Tasklet 모델을 사용하면서 Spring Batch에서 제공하는 Tasklet 인터페이스를 구현해야한다.
- Tasklet 구현클래스
  - SystemCommandTasklet
    - 시스템 명령어를 비동기적으로 실행하는 Tasklet이다.
    - 명령 속성에 수행해야할 명령어를 지정하여 사용할 수 있다.
    - 시스템 명령은 호출하는 스레드와 다른 스레드에 의해 실행되므로 프로세스 도중 타임아웃을 설정하고, 시스템 명령의 실행 스레드를 취소할 수 있다.
  - MethodInvokingTaskletAdapter:
    - POJO클래스의 특정 메소드를 실행하기 위한 태스클릿이다.
    - targetObject 속성에 대상 클래스의 빈을 지정하고, targetMethod속성에 실행할 메소드 이름을 지정한다.
    - POJO 클래스는 일괄 처리 종료 상태를 메소드의 반환 값으로 반환이 가능하지만, 이경우 사실은 ExitStatus를 반환값으로 설정해야한다.
    - 다른 타입의 값이 반환될 경우 반환값과 상관없이 "정상 종료(ExitStatus:COMPLETED)" 상태로 간주된다.
   
#### JpaPagingItemReader로 읽고, JpaItemWriter로 쓰기
- JpaPagingItemReader
  - Spring Batch에서 제공하는 ItemReader로 JPA를 사용하여 DB로부터 데이터를 페이지 단위로 읽는다.
  - JPA 기능활용 가능하다.
    - JPA 엔티티 기반 데이터 처리, 객체 매핑 자동화 등 JPA의 다양한 기능 활용
  - 쿼리 최적화
    - JPA 쿼리 기능을 사용하여 최적화된 데이터 읽기 가능
  - 커서 제어
    - JPA Criteria API를 사용하여 데이터 순회 제어
- JpaPagingItemReader 주요 구성 요소
  - EntityManagerFactory: JPA 엔티티 매니저 팩토리 설정
  - JpaQueryProvider: 데이터를 읽을 JPA 쿼리 제공
  - PageSize: 페이지 크기 설정
  - SkippableItemReader: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다
  - ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트 처리 가능
  - SaveStateCallback: 잠시 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리
- JpaItemWriter
  - Spring Batch에서 제공하는 ItemWriter 인터페이스 구현 클래스
  - 데이터를 JPA를 통해 데이터베이스에 저장하는 데 사용- 장점
    - ORM 연동: JPA를 통해 다양한 데이터베이스에 데이터 저장 가능
    - 객체 매핑: 엔티티 객체를 직접 저장하여 코드 간결성을 높일 수 있다.
    - 유연성: 다양한 설정을 통해 원하는 방식으로 데이터를 저장할 수 있다.
  - 단점
    - 설정 복잡성: JPA 설정 복잡
    - 오류 가능성: 설정 오류 시 데이터 손상 가능
- JpaItemWriter 구성요소
  - EntityManagerFactory: JPA ENtityManager 생성을 위한 팩토리 객체
  - JpaQueryProvider: 저장할 엔티티를 위한 JPA 쿼리를 생성하는 역할

#### 예제
- Entity
  ```java
  @Entity
  public class Customer {
      @Id
      @GeneratedValue(strategy = GenerationType.AUTO)
      private Long id;
  
      private String name;
      private Integer age;
      private String gender;
  
      public Customer() {}
      public Customer(Long id, String name, Integer age, String gender) {
          this.id = id;
          this.name = name;
          this.age = age;
          this.gender = gender;
      }
  
      public Long getId() { return id; }
      public String getName() { return name; }
      public Integer getAge() { return age; }
      public String getGender() { return gender; }
  }
  ```
- JpaPagingItemReader, JpaItemWriter, Step, Job 생성
  ```java
  @Configuration
  public class CustomerJpaJobConfig {  
      private final Logger log = LoggerFactory.getLogger(CustomerJpaPagingJobConfig.class);
      @Autowired
      private EntityManagerFactory entityManagerFactory;
  
      @Bean
      public JpaPagingItemReader<Customer> customerJpaPagingItemReader() throws Exception {
          String queryString = """
                  SELECT c
                  FROM Customer c
                  WHERE c.age > :age
                  ORDER BY id DESC
                  """;
          // 생성자를 이용한 방법
  //        JpaPagingItemReader<Customer> jpaPagingItemReader = new JpaPagingItemReader<>();
  //        jpaPagingItemReader.setQueryString(queryString);
  //        jpaPagingItemReader.setEntityManagerFactory(entityManagerfactory);
  //        jpaPagingItemReader.setPageSize(10);
  //        jpaPagingItemReader.setParameterValues(Collections.singletonMap("age", 20));
  //        return jpaPagingItemReader;
  
          // 빌더를 이용한 방법
          return new JpaPagingItemReaderBuilder<Customer>()
                  .name("customerJpaPagingItemReader")
                  .queryString(queryString)
                  .entityManagerFactory(entityManagerFactory)
                  .pageSize(10)
                  .parameterValues(Collections.singletonMap("age", 20))
                  .build();
      }
  
      @Bean
      public JpaItemWriter<Customer> customerJpaItemWriter() {
          return new JpaItemWriterBuilder<Customer>()
                  .entityManagerFactory(entityManagerFactory)
                  .usePersist(true)
                  .build();
      }
  
      @Bean
      public Step customerJpaPagingStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) throws Exception {
          log.info("------------------ Init customerJpaPagingStep -----------------");
  
          return new StepBuilder("customerJpaPagingStep", jobRepository)
                  .<Customer, Customer>chunk(10, transactionManager)
                  .reader(customerJpaPagingItemReader())
                  .processor(new CustomerItemProcessor())
                  .writer(customerJpaItemWriter())
                  .build();
      }
  
      @Bean
      public Job customerJpaPagingJob(Step customerJpaPagingStep, JobRepository jobRepository) {
          log.info("------------------ Init customerJpaPagingJob -----------------");
          return new JobBuilder()
                  .incrementer(new RunIdIncrementer())
                  .start(customerJpaPagingStep)
                  .build();
      }
  
  }
  ``` 
- ItemProcessor
  ```java
  public class CustomerItemProcessor implements ItemProcessor<Customer, Customer> {
      private final Logger log = LoggerFactory.getLogger(CustomerItemProcessor.class);  
      @Override
      public Customer process(Customer item) throws Exception {
          log.info("Item Processor ============ {}", item);
          return item;
      }
  }
  ```

#### CompositeItemProcessor 로 여러단계 걸쳐 데이터 Transform 하기
- CompositeItemProcessor
  - Spring Batch에서 제공하는 ItemProcessor 인터페이스로 구현하는 클래스
  - 여러개의 ItemProcessor를 하나의 Processor로 연결하여 여러 단계의 처리를 할 수 있다.
- 주요 구성 요소
  - deleagtes: 처리를 수행할 ItemProcessor 목록
  - TransactionAttribute: 트랜잭션 속성 설정
- 장점
  - 단계별 처리: 여러 단계로 나누어 처리를 수행하여 코드를 명확하고 이해하기 쉽게 만들 수 있다.
  - 재사용 가능성: 각 단계별 Processor를 재사용하여 다른 Job에서도 활용할 수 있다.
  - 유연성: 다양한 ItemProcessor를 조합하여 원하는 처리 과정을 구현할 수 있다.
- 단점
  - 설정 복잡성: 여러 개의 Processor를 설정하고 관리해야 하기 때문에 설정이 복잡해질 수 있다.
  - 성능 저하: 여러 단계의 처리 과정을 거치므로 성능이 저하될 수 있다.

#### 예제
- ItemProcessor 1 - name, gender 소문자 변경
  ```java
  public class LowerCaseItemProcessor implements ItemProcessor<Customer, Customer> {
      @Override
      public Customer process(Customer item) throws Exception {
          item.setName(item.getName().toLowerCase());
          item.setGender(item.getGender().toLowerCase());
          return item;
      }
  }
  ``` 
- ItemProcessor 2 - age + 20
  ```java
  public class After20YearsItemProcessor implements ItemProcessor<Customer, Customer> {
      @Override
      public Customer process(Customer item) throws Exception {
          item.setAge(item.getAge() + 20);
          return item;
      }
  }
  ```
- CompositeItemProcessor
  ```java
  @Bean
  public CompositeItemProcessor<Customer, Customer> compositeItemProcessor() {
      return new CompositeItemProcessorBuilder<Customer, Customer>()
              .delegates(List.of(
                      new LowerCaseItemProcessor(),
                      new After20YearsItemProcessor()
              ))
              .build();
  }
  ```

#### Custom ItemReader, ItemWriter
- QuerydslPagingItemReader
  - Querydsl은 SpringBatch에서 제공하는 공식 ItemReader가 아니기 때문에 AbstractpagingItemReader를 이용하여 Querydsl을 활용할 수 있도록 한다.
  - Querydsl을 활용하여 JPA 엔티티 추상화, 동적 쿼리를 지원할 수 있다.
- 예제
  ```java
  public class QuerydslPagingItemReader<T> extends AbstractPagingItemReader<T> {
      private EntityManager em;
      private final Function<JPAQueryFactory, JPAQuery<T>> querySupplier;
  
      private final Boolean alwaysReadFromZero;
  
      public QuerydslPagingItemReader(EntityManagerFactory entityManagerFactory, Function<JPAQueryFactory, JPAQuery<T>> querySupplier, int chunkSize) {
          this(ClassUtils.getShortName(QuerydslPagingItemReader.class), entityManagerFactory, querySupplier, chunkSize, false);
      }
  
      public QuerydslPagingItemReader(String name, EntityManagerFactory entityManagerFactory, Function<JPAQueryFactory, JPAQuery<T>> querySupplier, int chunkSize, Boolean alwaysReadFromZero) {
          super.setPageSize(chunkSize);
          setName(name);
          this.querySupplier = querySupplier;
          this.em = entityManagerFactory.createEntityManager();
          this.alwaysReadFromZero = alwaysReadFromZero;
  
      }
  
      @Override
      protected void doClose() throws Exception {
          if (em != null)
              em.close();
          super.doClose();
      }
  
      @Override
      protected void doReadPage() {
          initQueryResult();
  
          JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(em);
          long offset = 0;
          if (!alwaysReadFromZero) {
              offset = (long) getPage() * getPageSize();
          }
  
          JPAQuery<T> query = querySupplier.apply(jpaQueryFactory).offset(offset).limit(getPageSize());
  
          List<T> queryResult = query.fetch();
          for (T entity: queryResult) {
              em.detach(entity);
              results.add(entity);
          }
      }
  
      private void initQueryResult() {
          if (CollectionUtils.isEmpty(results)) {
              results = new CopyOnWriteArrayList<>();
          } else {
              results.clear();
          }
      }
  }
  ```
  - AbstrctPagingItemReader는 어댑터 패턴으로 상속 받는 쪽은 doReadPage 구현
  - 생성자
    - name: ItemReader를 구분하기 위한 이름이다.
    - entityManagerFactory: JPA를 이용하기 위해서 entityManagerFactory 를 전달한다.
    - Function<JPAQueryFactory, JPAQuery>: 이는 JPAQuery를 생성하기 위한 Functional Interface 이다.
    - 입력 파라미터로 JPAQueryFactory 를 입력으로 전달 받는다.
    - 반환값은 JPAQuery 형태의 queryDSL 쿼리가 된다.
    - chunkSize: 한번에 페이징 처리할 페이지 크기이다.
    - alwaysReadFromZero: 항상 0부터 페이징을 읽을지 여부를 지정한다. 만약 paging 처리된 데이터 자체를 수정하는경우 배치처리 누락이 발생할 수 있으므로 이를 해결하기 위한 방안으로 사용된다.
  - doClose
    - doClose는 기본적으로 AbstractPagingItemReader를 자체 구현되어 있지만 EntityManager자원을 해제하기 위해서 em.close() 를 수행한다.
  - doReadPage
    - JPAQueryFactory를 통해서 함수형 인터페이스로 지정된 queryDSL에 적용할 QueryFactory이다.
    - 만약 alwaysReadFromZero 가 false라면 offset과 limit을 계속 이동하면서 조회하도록 offset을 계산한다.
    - querySupplier.apply
      - 우리가 제공한 querySupplier에 JPAQueryFactory를 적용하여 JPAQuery를 생성하도록한다.
      - 페이징을 위해서 offset, limit을 계산된 offset과 pageSize (청크크기) 를 지정하여 페이징 처리하도록 한다.
    - fetch:
      - 결과를 패치하여 패치된 내역을 result에 담는다.
      - 이때 entityManager에서 detch하여 변경이 실제 DB에 반영되지 않도록 영속성 객체에서 제외시킨다.
  - initQueryResult
    - 매 페이징 결과를 반환할때 페이징 결과만 반환하기 위해서 초기화한다.
    - 만약 결과객체가 초기화 되어 있지 않다면 CopyOnWriteArrayList 객체를 신규로 생성한다.

- CustomItemWriter
  - CustomItemWriter는 Spring Batch에서 제공하는 기본 ItemWriter 인터페이스를 구현하여 직접 작성한 ItemWriter 클래스이다.
  - 기본 ItemWriter 클래스로는 제공되지 않는 특정 기능을 구현할 때 사용된다
  - 구성 요소
    - ItemWriter 인터페이스 구현: write() 메소드를 구현하여 원하는 처리를 수행한다.
    - 필요한 라이브러리 및 객체 선언: 사용할 라이브러리 및 객체를 선언한다.
    - 데이터 처리 로직 구현: write() 메소드에서 데이터 처리 로직을 구현한다.
  - 장점
    - 유연성: 기본 ItemWriter 클래스로는 제공되지 않는 특정 기능을 구현할 수 있다.
    - 확장성: 다양한 방식으로 데이터 처리를 확장할 수 있다.
    - 제어 가능성: 데이터 처리 과정을 완벽하게 제어할 수 있다.
  - 단점
    - 개발 복잡성: 기본 ItemWriter 클래스보다 개발 과정이 더 복잡하다.
    - 테스트 어려움: 테스트 작성이 더 어려울 수 있다.
    - 디버깅 어려움: 문제 발생 시 디버깅이 더 어려울 수 있다.
- 예제
  ```java
  @Component
  public class CustomItemWriter implements ItemWriter<Customer> {
      private final Logger log = LoggerFactory.getLogger(CustomItemWriter.class);  
      private final CustomService customService;
  
      public CustomItemWriter(CustomService customService) {
          this.customService = customService;
      }
  
      @Override
      public void write(Chunk<? extends Customer> chunk) throws Exception {
          for (Customer customer: chunk) {
              log.info("Call Porcess in CustomItemWriter...");
              customService.processToOtherService(customer);
          }
      }
  }
  ```

#### 출처
- https://devocean.sk.com/experts/techBoardDetail.do?ID=166694&boardType=experts&page=&searchData=&subIndex=&idList=&searchText=&techType=&searchDataSub=&searchDataMain=&writerID=kido&comment=
- https://devocean.sk.com/experts/techBoardDetail.do?ID=166902&boardType=experts&page=&searchData=&subIndex=&idList=&searchText=&techType=&searchDataSub=&searchDataMain=&writerID=kido&comment=
- https://devocean.sk.com/experts/techBoardDetail.do?ID=167030&boardType=experts&page=&searchData=&subIndex=&idList=&searchText=&techType=&searchDataSub=&searchDataMain=&writerID=kido&comment=
