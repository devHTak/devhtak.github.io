---
layout: post
title: Spring Batch Architecture
summary: Spring Batch
author: devhtak
date: '2025-03-17 21:41:00 +0900'
category: Spring
---
### Spring Batch 

#### 스프링 배치 시작하기
- 의존성 추가
  - spring-boot-starter-batch 라는 의존성 추가
- 배치 기동시키기
  - @EnableBatchProcessing 어노테이션을 이용하면 기본적으로 스프링 배치 모드로 동작
  - Spring Boot 3.0 이상부터는 해당 애노테이션 없이도 동작 가능
- DataSource 구성

#### 단순 예제
- Tasklet 생성
```java
public class GreetingTask implements Tasklet, InitializingBean {

    private Logger log = LoggerFactory.getLogger(GreetingTask.class);

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        log.info("========TASK EXECUTE========");
        log.info("Greeting Task: {}, {}", contribution, chunkContext);

        return RepeatStatus.FINISHED; // FINISHED(Taskleet 작업 종료), CONTRNUABLE(계속해서 태스크 수행), continuable(조건에 따라 종료/지속할지 결정)
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("========AFTER PROPERTIES SETS()===========");
    }
}
```
- Task 빈등록 및 Step, Job 생성
```java
@Configuration
public class BasicTaskJobConfiguration {

    private final Logger log = LoggerFactory.getLogger(BasicTaskJobConfiguration.class);

    @Bean
    public Tasklet greetingTasklet() {
        return new GreetingTask();
    }

    @Bean
    public Step step(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
        log.info("Init MyStep");

        return new StepBuilder("myStep", jobRepository)
                .tasklet(greetingTasklet(), transactionManager)
                .build();
    }

    @Bean
    public Job myJob(Step step, JobRepository jobRepository) {
        log.info("Init MyJob");

        return new JobBuilder("myJob", jobRepository)
                .incrementer(new RunIdIncrementer()) // Job이 지속적으로 실행될 때 unique하게 구분할 수 있도록 함(Job 아이디 증가)
                .start(step)
                .build();
    }
}
```
- 실행 결과
```
2025-03-17T13:30:05.796+09:00  INFO 1598 --- [spring-batch-example] [           main] com.example.tasklet.GreetingTask         : ========AFTER PROPERTIES SETS()===========
2025-03-17T13:30:05.810+09:00  INFO 1598 --- [spring-batch-example] [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2025-03-17T13:30:05.945+09:00  INFO 1598 --- [spring-batch-example] [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection conn0: url=jdbc:h2:mem:1b216e94-7119-4e86-9b31-9aa98e274ecc user=SA
2025-03-17T13:30:05.946+09:00  INFO 1598 --- [spring-batch-example] [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2025-03-17T13:30:06.018+09:00  INFO 1598 --- [spring-batch-example] [           main] c.e.config.BasicTaskJobConfiguration     : Init MyStep
2025-03-17T13:30:06.048+09:00  INFO 1598 --- [spring-batch-example] [           main] c.e.config.BasicTaskJobConfiguration     : Init MyJob
2025-03-17T13:30:06.162+09:00  INFO 1598 --- [spring-batch-example] [           main] c.example.SpringBatchExampleApplication  : Started SpringBatchExampleApplication in 1.436 seconds (process running for 1.719)
2025-03-17T13:30:06.164+09:00  INFO 1598 --- [spring-batch-example] [           main] o.s.b.a.b.JobLauncherApplicationRunner   : Running default command line with: []
2025-03-17T13:30:06.203+09:00  INFO 1598 --- [spring-batch-example] [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=myJob]] launched with the following parameters: [{'run.id':'{value=1, type=class java.lang.Long, identifying=true}'}]
2025-03-17T13:30:06.222+09:00  INFO 1598 --- [spring-batch-example] [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [myStep]
2025-03-17T13:30:06.228+09:00  INFO 1598 --- [spring-batch-example] [           main] com.example.tasklet.GreetingTask         : ========TASK EXECUTE========
2025-03-17T13:30:06.228+09:00  INFO 1598 --- [spring-batch-example] [           main] com.example.tasklet.GreetingTask         : Greeting Task: [StepContribution: read=0, written=0, filtered=0, readSkips=0, writeSkips=0, processSkips=0, exitStatus=EXECUTING], ChunkContext: attributes=[], complete=false, stepContext=SynchronizedAttributeAccessor: [], stepExecutionContext={batch.version=5.1.3, batch.taskletType=com.example.tasklet.GreetingTask, batch.stepType=org.springframework.batch.core.step.tasklet.TaskletStep}, jobExecutionContext={batch.version=5.1.3}, jobParameters={run.id=1}
2025-03-17T13:30:06.239+09:00  INFO 1598 --- [spring-batch-example] [           main] o.s.batch.core.step.AbstractStep         : Step: [myStep] executed in 16ms
2025-03-17T13:30:06.242+09:00  INFO 1598 --- [spring-batch-example] [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=myJob]] completed with the following parameters: [{'run.id':'{value=1, type=class java.lang.Long, identifying=true}'}] and the following status: [COMPLETED] in 31ms
```

#### 스프링 배치 스키마 구조
- 스프링 배치를 수행하면 자동으로 배치를 위한 스키마가 형성된다.
  - <img width="777" alt="image" src="https://github.com/user-attachments/assets/fcfa3f5b-9f7d-40de-a388-05dea67f0687" />
  - 출처: https://docs.spring.io/spring-batch/reference/schema-appendix.html

##### Table
- BATCH_JOB_INSTANCE
  - 스키마 중 가장 기본이 되는 배치 잡 인스턴스 테이블
  - 배치가 수행되면 Job이 생성되고 해당 Job 인스턴스에 대해서 관련된 모든 정보를 가진 최상위 테이블
- BATCH_JOB_EXCUTION_PARAMS
  - JobParameter 에 대한 정보를 저장하는 테이블
  - Key/Value 형태로 Job에 전달되며 Job이 실행될 때 전달된 파라미터 정보를 저장한다.
  - 각 파라미터는 IDENTIFYING이 true로 설정되며 JobParameter 생성 시 유니크한 값으로 사용된 경우라는 의미
- BATCH_JOB_EXECUTION
  - JobExecution과 관련된 모든 정보를 저장
  - Job이 매번 실행될 때, JobExecution이라는 새로운 객체가 있으며, 이 테이블에 새로운 row로 생성
- BATCH_STEP_EXECUTION
  - StepExecution과 관련된 정보
  - BATCH_JOB_EXECUTION 테이블과 유사하며 생성된 각 JobExecution에 대한 단계당 항목이 하나 이상 저장된다.
- BATCH_JOB_EXECUTION_CONTEXT
  - Job의 ExecutionContext에 대한 모든 정보를 저장
    - ExecutionContext는 배치 처리 중 데이터를 저장하고 공유하는데 사용되는 구성요소
    - 데이터 저장 및 공유 / Job 재시작 시 상태 복구 / 지속 가능한 상태 유지 로 3가지 역할을 한다.
  - 매 JobExecution 마다 정확히 하나의 JobExecutionContext를 가진다. 특정 작업 실행에 필요한 모든 작업 수준 데이터가 포함
  - 해당 데이터는 일반적으로 실패 후 중단된 부분부터 시작될 수 있도록 실패 후 검색해야하는 상태를 나타낸다.
- BATCH_STEP_EXECUTION_CONTEXT
  - Step의 ExecutionContext에 대한 모든 정보를 저장
  - 매 StepExecution마다 ExecutionContext가 있다. 특정 step execution에 대해 저장될 필요가 있는 모든 데이터가 저장된다.
  - 해당 데이터는 일반적으로 JobInstance가 중단된 위치에서 시작할 수 있도록 실패 후 검색해야하는 상태를 나타낸다.

##### Sequence Table
- 스프링 배치는 기본적으로 시퀀스 테이블이 존재하며 해당 시퀀스를 배치가 할당하여 중복되지 않도록 한다.
- BATCH_JOB_SEQ
  - 배치 잡에 대한 시퀀스 테이블
- BATCH_JOB_EXECUTION_SEQ
  - 배치 잡 Execution의 시퀀스 테이블
- BATCH_STEP_EXECUTION_SEQ
  - 배치 스텝 Execution의 시퀀스 데이틀
 
#### 스프링 배치 아키텍처
- 아키텍처
  <img width="754" alt="image" src="https://github.com/user-attachments/assets/6b13d6a6-2ede-461e-94f5-ad28c964d9e3" />
- 스프링 배치 흐름
  <img width="804" alt="image" src="https://github.com/user-attachments/assets/6a9f3040-9329-48c0-9e12-d5deaeb55b38" />
  - 출처: https://devocean.sk.com/experts/techBoardDetail.do?page=&boardType=experts&query=&ID=166690&searchData=&subIndex=&searchText=&techType=&searchDataSub=&searchDataMain=&comment=
- Job
  - Spring Batch에서 일괄 적용을 위한 일련의 프로세스를 요악하는 단일 실행 단위
- Step
  - Job을 구성하는 처리단위로 하나의 Job의 여러 Step을 할당할 수 있으며 재사용, 병렬화, 조건 분기등을 수행할 수 있다.
  - Step은 tasklet 모델 / chunk 모델의 구현체가 탐재되어 실행
- JobLauncher
  - Job을 수행하기 위한 인터페이스로 JobLauncher는 사용자에 의해 직접 수행된다.
  - 자바 커맨드를 통해 CommandLineJobRunner를 실행하여 단순하게 배치 프로세스가 수행될 수 있다.
- ItemReader
  - 청크단위 모델에서 사용하며, 소스 데이터를 읽어 들이는 역할 수행
- ItemProcessor
  - 읽어들인 청크 데이터를 처리, 데이터 변환을 수행하거나 데이터를 정제하는 등의 역할을 담당
  - 옵션으로 필요없다면 사용하지 않다도 된다.
- ItemWriter
  - 청크 데이터를 읽어들였거나 처리된 데이터를 실제 쓰기작업 담당
  - 데이터베이스로 저장하거나, 수정하는 역할을 할 수 있고, 파일로 처리 결과를 출력할 수 있다.
- Tasklet
  - 단순하고 유연하게 배치 처리를 수행하는 테스크를 수행
- JobRepository
  - Job과 Step의 상태를 관리한다.
  - 스프링배치에서 사용하는 테이블 스키마를 기반으로 상태정보를 저장하고 관리

##### 저장 정보
- JobInstance
  - JobInstance는 잡 이름과 전달 파라미터를 정의한다.
  - Job이 중단되는 경우 다음 실행할때 중단 이후부터 실행하도록 지원한다. (ExecutionContext)
  - Job이 재실행을 지원하지 않는경우, 혹은 성공적으로 처리된 경우 배치를 재실행 한다면 중복 수행되지 않도록 종료한다.
- JobExecution
  - JobExecution은 잡의 물리적인 실행을 나타낸다.
  - JobInstance와 달리 동일한 Job이 여러번 수행될 수 있다.
  - 그러므로 JobInstance 와 JobExecution은 1:N 관계가 된다.
- ExecutionContext
  - ExecutionContext는 각각의 JobExecution 에서 처리 단계와 같은 메타 정보들을 공유하는 영역이다.
  - ExecutionContext는 주로 스프링배치가 프레임워크 상태를 기록하는데 사용하며, 또한 애플리케이션에서 ExecutionContext에 액세스 하는 수단도 제공된다.
  - ExecutionContext에 저장되는 객체는 java.io.Serialized를 구현하는 클래스이어야 한다.
- StepExecution
  - StepExecution은 Step을 물리적인 실행을 나타낸다.
  - Job은 여러 Step을 수행하므로 1:N 관계가 된다.
- ExecutionContext
  - Step내부에 데이터를 공유해야하는 공유 영역이다.
  - 데이터의 지역화 관점에서 여러 단계에 공유 할 필요가 없는 정보는 Job내 ExecutionContext를 이용하는 대신에, Step 단계 내의 ExecutionContext를 사용해야한다.
  - StepExecutionContext에 저장되는 데이터는 반드시 java.io.Serializable 를 구현해야한다.
- JobRepository
  - JobExecution과 StepExecution등과 같이 배치 실행정보나 상태, 결과정보들이 데이터베이스에 저장될 필요가 있으며 이를 처리하는 것이 JobRepository이다.
  - 즉 스프링배치를 수행하기 위해서 이를 저장할 데이터베이스가 필요하다.
  - 이렇게 저장된 정보를 활용하여 스프링배치는 배치 잡을 재실행 하거나, 정지된 상태 후부터 수행할 수 있는 수단을 제공하게 된다.

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

#### 출처
- https://devocean.sk.com/experts/techBoardDetail.do?page=&boardType=experts&query=&ID=166164&searchData=&subIndex=&searchText=&techType=&searchDataSub=&searchDataMain=&comment=
- https://devocean.sk.com/experts/techBoardDetail.do?page=&boardType=experts&query=&ID=166690&searchData=&subIndex=&searchText=&techType=&searchDataSub=&searchDataMain=&comment=
