---
layout: post
title: Spring Batch
summary: Spring Batch
author: devhtak
date: '2021-11-06 21:41:00 +0900'
category: Spring
---

#### Spring Batch

- 로깅/추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 건너뛰기, 리소스 관리 등 대용량 레코드 처리에 필수적 기능 제공
- 최적화 및 파티셔닝 기술을 통해 대용량 및 고성능 배치 작업을 가능하게 한다
- 배치가 실패하여 작업 재시작을 하면 처음부터가 아닌 실패한 시점부터 실행하게 된다.
- 중복 실행을 막기 위해 성공한 이력이 있는 Batch 는 동일한 Parameters로 실행 시 Exception 발생한다

```
Spring Batch는 Scheduler가 아니다.
Batch Job을 관리하지만, Job을 구동하거나 발생시키는 기능은 지원하지 않기 때문에, Quartz, Scheduler, Jenkins 등 을 사용해야 한다.
```

#### Spring Batch 용어

- Job
  - Job은 배치처리 과정을 하나의 단위로 만들어 놓은 객체
  - 배치처리 과정에 있어 전체 계층 최상단에 위치하고 있습니다.

- JobInstance
  - Job의 실행의 단위
  - Job을 실행시키게 되면 하나의 JobInstance가 생성되게 됩니다. 
    - EX) 1월 1일 실행, 1월 2일 실행을 하게 되면 각각의 JobInstance가 생성되며 1월 1일 실행한 JobInstance가 실패하여 다시 실행을 시키더라도 이 JobInstance는 1월 1일에 대한 데이터만 처리하게 됩니다.

- JobParameters
  - Job에 대한 실행단위인 JobInstance를 JobParameters 객체를 통해 구분
  - 추가로 개발자 JobInstacne에 전달되는 매개변수 역할도 하고 있습니다.
  - String, Double, Long, Date 4가지 형식만을 지원하고 있습니다.

- JobExecution
  - JobInstance에 대한 실행 시도에 대한 객체입니다.
  - 1월 1일에 실행한 JobInstacne가 실패하여 재실행을 하여도 동일한 JobInstance를 실행시키지만 이 2번에 실행에 대한 JobExecution은 개별로 생기게 됩니다. 
  - JobExecution는 이러한 JobInstance 실행에 대한 상태,시작시간, 종료시간, 생성시간 등의 정보를 담고 있다.

- Step
  - Job의 배치처리를 정의하고 순차적인 단계를 캡슐화
  - Job은 최소한 1개 이상의 Step을 가져야 하며 Job의 실제 일괄 처리를 제어하는 모든 정보가 들어있다

- StepExecution
  - JobExecution과 동일하게 Step 실행 시도에 대한 객체
  - Job이 여러개의 Step으로 구성되어 있을 경우 이전 단계의 Step이 실패하게 되면 다음 단계가 실행되지 않음으로 실패 이후 StepExecution은 생성되지 않습니다. StepExecution 또한 JobExecution과 동일하게 실제 시작이 될 때만 생성됩니다. StepExecution에는 JobExecution에 저장되는 정보 외에 read 수, write 수, commit 수, skip 수 등의 정보들도 저장이 됩니다.

- ExecutionContext
  - Job에서 데이터를 공유 할 수 있는 데이터 저장소
  - ExecutionContext는 JobExecutionContext, StepExecutionContext 2가지 종류로 이 두가지는 지정되는 범위가 다르다
    - JobExecutionContext의 경우 Commit 시점에 저장되는 반면 StepExecutionContext는 실행 사이에 저장
    - ExecutionContext를 통해 Step간 Data 공유가 가능하며 Job 실패시 ExecutionContext를 통한 마지막 실행 값을 재구성 할 수 있다

- JobRepository
  - 모든 배치 처리 정보를 담고있는 매커니즘
  - Job이 실행되게 되면 JobRepository에 JobExecution과 StepExecution을 생성하게 되며 JobRepository에서 Execution 정보들을 저장하고 조회하며 사용

- JobLauncher
  - Job과 JobParameters를 사용하여 Job을 실행하는 객체

- ItemReader
  - Step에서 Item을 읽어오는 인터페이스
  - ItemReader에 대한 다양한 인터페이스가 존재하며 다양한 방법으로 Item을 읽어 올 수 있다

- ItemWriter
  - ItemWriter는 처리 된 Data를 Writer 할 때 사용
  - Writer는 처리 결과물에 따라 Insert가 될 수도 Update가 될 수도 Queue를 사용한다면 Send가 될 수 있다
  - Writer 또한 Read와 동일하게 다양한 인터페이스가 존재하며 기본적으로 Item을 Chunk로 묶어 처리

- ItemProcessor
  - Reader에서 읽어온 Item을 데이터를 처리하는 역할을 하고 있다. 
  - Processor는 배치를 처리하는데 필수 요소는 아니며 Reader, Writer, Processor 처리를 분리하여 각각의 역할을 명확하게 구분

#### Job Example

- Job은 여러가지 Step의 모음으로 구성되어 있으며 순차적으로 Step을 수행하며 Batch를 수행한다.
- 단일 Step 구성 예제
  ```java
  @Slf4j
  @Configuration
  @EnableBatchProcessing
  public class SingleStepExampleConfig {

      @Autowired
      public JobBuilderFactory jobBuilderFactory;
      @Autowired
      public StepBuilderFactory stepBuilderFactory;

      @Bean
      public Job exampleJob() {
          Job exampleJob = jobBuilderFactory.get("exampleSingleStepJob")
                  .start(step())
                  .build();

          return exampleJob;
      }

      @Bean
      public Step step() {
          return stepBuilderFactory.get("singleStep")
                  .tasklet((contribution, chunkContext) -> {
                      log.info("Single Step!");
                      return RepeatStatus.FINISHED;
                  }).build();
      }
  }
  ```
  
- 다중 step 사용 예제
  ```java
  @Slf4j
  @Configuration
  @EnableBatchProcessing
  public class MultiStepExampleConfig {

      @Autowired
      public JobBuilderFactory jobBuilderFactory;

      @Autowired
      public StepBuilderFactory stepBuilderFactory;

      @Bean
      public Job exampleJob() {
          Job exampleMultiStepJob = jobBuilderFactory.get("exampleMultiStepJob")
                  .start(startStep())
                  .next(nextStep())
                  .next(lastStep())
                  .build();

          return exampleMultiStepJob;

      }

      @Bean
      public Step lastStep() {
          return stepBuilderFactory.get("lastStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("last step!");
                      return RepeatStatus.FINISHED;
                  })).build();
      }
      
      @Bean
      public Step nextStep() {
          return stepBuilderFactory.get("nextStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("next step!");
                      return RepeatStatus.FINISHED;
                  })).build();
      }
      
      @Bean
      public Step startStep() {
          return stepBuilderFactory.get("startStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("Start Step!");
                      return RepeatStatus.FINISHED;
                  })).build();
      }
  }
  ```

- Flow를 통한 Step 구성
  ```java
  @Slf4j
  @Configuration
  @EnableBatchProcessing
  public class FlowStepExampleConfig {

      @Autowired
      public JobBuilderFactory jobBuilderFactory;

      @Autowired
      public StepBuilderFactory stepBuilderFactory;


      @Bean
      public Job exampleJob() {
          Job flowStepJob = jobBuilderFactory.get("flowStepJob")
                  .start(startStep())
                          .on("FAILED") // start step의 ExistStatus.FAILED인 경우
                          .to(failOverStep())// failover step 실행
                          .on("*") // failover step의 결과와 상관없이
                          .to(writeStep())// write step 실행
                          .on("*") // write step 결과와 상관 없이
                          .end() // flow 종료
                  .from(startStep())
                          .on("COMPLETED") // startStep에서 FAILED가 아닌 COMPLETED인 경우
                          .to(processStep())
                          .on("*")
                          .to(writeStep())
                          .on("*")
                          .end()

                  .from(startStep())
                          .on("*")// startStep이 FAILED, COMPLETED가 아닌 경우
                          .to(writeStep())
                          .end()

                  .build();
          return flowStepJob;
      }
      
      @Bean
      public Step processStep() {
          return stepBuilderFactory.get("processStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("process step");
                      return RepeatStatus.FINISHED;
                  })).build();
      }
      
      @Bean
      public Step writeStep() {
          return stepBuilderFactory.get("writeStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("Write Step");
                          return RepeatStatus.FINISHED;
                  })).build();
      }

      @Bean
      public Step failOverStep() {
          return stepBuilderFactory.get("nextStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("FAILOVER STEP");
                      return RepeatStatus.FINISHED;
                  })).build();
      }
      
      @Bean
      public Step startStep() {
          return stepBuilderFactory.get("startFlowStep")
                  .tasklet(((stepContribution, chunkContext) -> {
                      log.info("start step!");
                      String result = "COMPLETED"; // COMPLETED, FAILED, UNKNOWN

                      // Flow 에서 on 은 RepeatStatus가 아닌 ExitStatus를 바라본다
                      if(result.equals("COMPLETED")) {
                          stepContribution.setExitStatus(ExitStatus.COMPLETED);
                      } else if(result.equals("FAILED")) {
                          stepContribution.setExitStatus(ExitStatus.FAILED);
                      } else if(result.equals("UNKNOWN")) {
                          stepContribution.setExitStatus(ExitStatus.UNKNOWN);
                      }
                      return RepeatStatus.FINISHED;
                  })).build();
      }

  }
  ```
  
#### Step 구성

- tasklet
  - Functional Interface로 실패를 알리기 위해 예외를 throw할 때까지 execute를 반복적으로 호출한다
  - lambda를 사용하여 구현하는 방법, Tasklet, StepExecutionListner 구현체 사용, MethodInvokingTaskletAdapter 를 통해 실행할 수 있다.
    ```java
    @Bean
    public Step taskStep() {
        return stepBuilderFactory().get("taskletStep")
            .tasklet(myTasklet()).build();
    }
    
    @Bean
    public MethodInvokingTaskletAdapter myTasklet() {
        MethodInvokingTaskletAdapter adapter = new MethodInvokingTaskletAdapter();
        adapter.setTargetObject(customerService);
        adapter.setTargetMethod("buisinessLogic");
        
        return adapter;
    }
    ```

- chunk
  - 처리되는 commit row 수를 의미
  - Batch 처리에서 커밋되는 row 수는 chunk 단위로 Transaction을 수행하기 때문에 실패 시 chunk 단위 만큼 rollback하게 된다.
  - chunk 시나리오
    - 읽기(read) - database에서 배치처리할 데이터를 읽어온다.
    - 처리(processing) - 읽어온 데이터를 가공, 처리한다 (필수X)
    - 쓰기(writing) - 가공, 처리한 데이터를 database에 저장
  
  - chunk size
    ```
    Setting a fairly large page size and using a commit interval that matches the page size should provide better performance.
    페이지 크기를 상당히 크게 설정하고 페이지 크기와 일치하는 커밋 간격을 사용하면 성능이 향상된다.
    ```
    - read 쿼리 수행 시 1번의 transaction을 위해 두 설정의 값을 일치 시키는게 가장 좋은 성능 향상 방법이다.

#### Step 설정

- startLimit
  ```java
  @Bean
  @JobScope
  public Step step() throws Exception {
      return stepBuilderFactory.get("StepEx")
              .startLimit(3)
              .<Member, Member>chunk(10)
              .reader(reader(null))
              .processor(processor(null))
              .writer(writer(null))
              .build();
  }
  ```
  - 해당 Step 이 실패 후 재시작 가능 횟수를 의미
  - startLimit 이후 실행은 Exception이 발생한다.

- skip
  ```java
  @Bean
  @JobScope
  public Step Step() throws Exception {
      return stepBuilderFactory.get("Step")
              .<Member,Member>chunk(10)
              .reader(reader(null))
              .processor(processor(null))
              .writer(writer(null))
              .faultTolerant()
              .skipLimit(1) // skip 허용 횟수, 해당 횟수 초과시 Error 발생, Skip 사용시 필수 설정
              .skip(NullPointerException.class)// NullPointerException에 대해선 Skip
              .noSkip(SQLException.class) // SQLException에 대해선 noSkip
              //.skipPolicy(new CustomSkipPolilcy) // 사용자가 커스텀하며 Skip Policy 설정 가능
              .build();
  }
  ```
  
- retry
  ```java
  @Bean
  @JobScope
  public Step Step() throws Exception {
      return stepBuilderFactory.get("Step")
              .<Member,Member>chunk(10)
              .reader(reader(null))
              .processor(processor(null))
              .writer(writer(null))
              .faultTolerant()
              .retryLimit(1) //retry 횟수, retry 사용시 필수 설정, 해당 Retry 이후 Exception시 Fail 처리
              .retry(SQLException.class) // SQLException에 대해선 Retry 수행
              .noRetry(NullPointerException.class) // NullPointerException에 no Retry
              //.retryPolicy(new CustomRetryPolilcy) // 사용자가 커스텀하며 Retry Policy 설정 가능
              .build();
  }
  ```

- noRollback
  ```java
  @Bean
  @JobScope
  public Step Step() throws Exception {
      return stepBuilderFactory.get("Step")
              .<Member,Member>chunk(10)
              .reader(reader(null))
              .processor(processor(null))
              .writer(writer(null))
              .faultTolerant()
              .noRollback(NullPointerException.class) // NullPointerException 발생  rollback이 되지 않게 설정
              .build();
  }
  ```

### @JobScope, @StepScode

- @JobScope
  - Step 선언문에 사용 가능

- @StepScope
  - Step을 구성하는 ItemREader, ItemProcessor, ItemWriter에 사용

- 특징
  - singleton 패턴이 아닌 annotation이 명시된 메소드의 실행 시점에 bean이 생성
  - @JobScope, @StepScope 빈이 생성될 때 JobParameter가 생성되기 때문에 JobParameter 사용하기 위해 scope을 지정해주어야 한다.
    - 유연한 설계, 병렬 실행을 위해 LateBinding을 하여 JobParameter를 비즈니스 로직 단계에서 할당

- JobLauncher 로 실행
  ```java
  @Configuration
  @Slf4j
  public class JobScheduler {

      @Autowired
      private JobLauncher jobLauncher;

      @Autowired
      private Job ExampleJob;

      @Scheduled(cron = "1 * * * * *")
      public void jobSchduled() throws JobParametersInvalidException, JobExecutionAlreadyRunningException,
          JobRestartException, JobInstanceAlreadyCompleteException {

          Map<String, JobParameter> jobParametersMap = new HashMap<>();

          SimpleDateFormat format1 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");
          Date time = new Date();

          String time1 = format1.format(time);

          jobParametersMap.put("date",new JobParameter(time1));

          JobParameters parameters = new JobParameters(jobParametersMap);

          JobExecution jobExecution = jobLauncher.run(ExampleJob, parameters);

          while (jobExecution.isRunning()) {
              log.info("...");
          }

          log.info("Job Execution: " + jobExecution.getStatus());
          log.info("Job getJobConfigurationName: " + jobExecution.getJobConfigurationName());
          log.info("Job getJobId: " + jobExecution.getJobId());
          log.info("Job getExitStatus: " + jobExecution.getExitStatus());
          log.info("Job getJobInstance: " + jobExecution.getJobInstance());
          log.info("Job getStepExecutions: " + jobExecution.getStepExecutions());
          log.info("Job getLastUpdated: " + jobExecution.getLastUpdated());
          log.info("Job getFailureExceptions: " + jobExecution.getFailureExceptions());
      }
  }
  ```

- JobParameters 사용
  ```java
  /**
   * 전체 금액이 10,000원 이상인 회원들에게 1,000원 캐시백을 주는 배치
   */

  @Slf4j
  @Configuration
  @EnableBatchProcessing
  public class ExampleJobConfig {

      @Autowired public JobBuilderFactory jobBuilderFactory;
      @Autowired public StepBuilderFactory stepBuilderFactory;
      @Autowired public EntityManagerFactory entityManagerFactory;

      @Bean
      public Job ExampleJob() throws Exception {

          Job exampleJob = jobBuilderFactory.get("exampleJob")
                  .start(Step())
                  .build();

          return exampleJob;
      }

      @Bean
      @JobScope
      public Step Step() throws Exception {
          return stepBuilderFactory.get("Step")
                  .<Member,Member>chunk(10)
                  .reader(reader(null))
                  .processor(processor(null))
                  .writer(writer(null))
                  .build();
      }

      @Bean
      @StepScope
      public JpaPagingItemReader<Member> reader(@Value("#{jobParameters[date]}")  String date) throws Exception {

          log.info("jobParameters value : " + date);

          Map<String,Object> parameterValues = new HashMap<>();
          parameterValues.put("amount", 10000);

          return new JpaPagingItemReaderBuilder<Member>()
                  .pageSize(10)
                  .parameterValues(parameterValues)
                  .queryString("SELECT p FROM Member p WHERE p.amount >= :amount ORDER BY id ASC")
                  .entityManagerFactory(entityManagerFactory)
                  .name("JpaPagingItemReader")
                  .build();
      }

      @Bean
      @StepScope
      public ItemProcessor<Member, Member> processor(@Value("#{jobParameters[date]}")  String date){

          return new ItemProcessor<Member, Member>() {
              @Override
              public Member process(Member member) throws Exception {

                  log.info("jobParameters value : " + date);

                  //1000원 추가 적립
                  member.setAmount(member.getAmount() + 1000);

                  return member;
              }
          };
      }

      @Bean
      @StepScope
      public JpaItemWriter<Member> writer(@Value("#{jobParameters[date]}")  String date){

          log.info("jobParameters value : " + date);

          return new JpaItemWriterBuilder<Member>()
                  .entityManagerFactory(entityManagerFactory)
                  .build();
      }
  }
  ```

#### Spring Meta Table

![image](https://user-images.githubusercontent.com/42403023/140601053-91132dd6-7a26-4ff3-a6af-828293cdd205.png)

- Spring Batch 에는 6개의 meta table 과 3개의 sequence table이 존재한다.
  - Spring Batch Job이 실행될 때마다 실행된 job에 대한 정보 저장 목적

- sequence
  - BATCH_JOB_INSTANCE, BATCH_JOB_EXECUTION및 BATCH_STEP_EXECUTION의 Primary Key는 시퀀스에 의해 생성

- table
  - BATCH_JOB_INSTANCE
    - JobInstance에 관련된 모든 정보 포함
    - 전체 계층 구조의 최상위 역할
    
  - BATCH_JOB_EXECUTION_PARAMS
    - Job을 실행 시킬 때 사용했던 JobParameters에 대한 정보 저장
    
  - BATCH_JOB_EXECUTION
    - JobExcution에 관련된 모든 정보 저장
    - JobInstance가 실행 될 때마다 시작시간, 종료시간, 종료코드 등 다양한 정보를 가지고 있다.
    
  - BATCH_STEP_EXECUTION
    - StepExecution에 대한 정보 저장
    - STEP을 EXECUTION 정보인 읽은 수, 커밋 수, 스킵 수 등 다양한 정보를 추가로 담고 있다.

  - BATCH_JOB_EXECUTION_CONTEXT
    - JobExecution의ExecutionContext 정보 저장
    - JobInstance가 실패 시 중단된 위치에서 다시 시작할 수 있는 정보를 저장하고 있다.

  - BATCH_STEP_EXECUTION_CONTEXT
    - tepExecution의 ExecutionContext 정보 저장
    - JobInstance가 실패 시 중단된 위치에서 다시 시작할 수 있는 정보를 저장하고 있다.


#### 출처

- https://khj93.tistory.com/entry/Spring-Batch%EB%9E%80-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0
