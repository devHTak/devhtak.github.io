---
layout: post
title: Spring Batch Flow
summary: Spring Batch
author: devhtak
date: '2025-03-18 21:41:00 +0900'
category: Spring
---

#### Flow Controller
- 배치 수행 Flow control 은 여러 Step을 정의하고 조건에 따라 순서대로 실행하거나 특정 step을 건너뛸 수 있도록 하는 기능
- FlowBuilder API를 사용하여 설정
  - next: 현재 Step이 성공적으로 종료되면 다음 Step으로 이동한다. next는 계속해서 추가 될 수 있으며, start --> next --> next ... 순으로 진행되도록 한다.
  - from: 특정 Step에서 현재 Step으로 이동한다.
  - on: 특정 ExitStatus에 따라 다음 Step을 결정한다.
  - to: 특정 Step으로 이동한다.
  - stop: 현재 Flow를 종료한다.
  - end: FlowBuilder를 종료한다.
- from/on/to/stop 예제 -> on에 조건이 일치하는 경우 to에 해당하는 step이 실행
  ```java
  @Bean
  public Job job() {  
      return jobBuilderFactory.get("job")
          .start(step1())
          .on("FAILED").to(step3()) // step1이 "FAILED"면(Exception 발생) step3 수행
          //.on("FAILED").stop() // step1이 "FAILED"면(Exception 발생) 배치 작업 정지
          .from(step1()).on("COMPLETED").to(step2()) //step1이 "COMPLETED"면(RepeatStatus.FINISHED) step2 수행
          .end()
          .build();
  }
  ```

#### Job/Step 마다 시작/종료 시 특정 작업 수행
##### JobExecutionListener
- JobExecutionListener는 Job이 시작/종료 할때마다 특정 이벤트를 수행하고 싶은경우 사용된다.
- JobBuilder를  정의할때 listener에 jobExecutionListener를 등록하여 사용
- 예제
  ```java
  @Slf4j
  @Configuration
  public class JobListenerConfig {  
      @Bean
      public JobExecutionListener jobExecutionListener() {
          return new JobExecutionListener() {
              @Override
              public void beforeJob(JobExecution jobExecution) {
                  log.info(" >>>>>> Before job: Job {} is starting...", jobExecution.getJobInstance().getJobName());
              }
  
              @Override
              public void afterJob(JobExecution jobExecution) {
                  log.info(" >>>>>> After job: Job {} is finished.", jobExecution.getJobInstance().getJobName());
              }
          };
      }
  }
  ```

##### StepExecutionListener
- StepListener은 Step이 시작/종료 할때마다 특정 이벤트를 수행하고 싶은경우 사용된다.
- StepBuilder를 정의할때 listener에 stepExecutionListener를 등록하면 된다.
- 예제
  ```java
  @Slf4j
  @Configuration
  public class StepListenerConfig {
      @Bean
      public StepExecutionListener stepExecutionListener() {
          return new StepExecutionListener() {
              @Override
              public void beforeStep(StepExecution stepExecution) {
                  log.info("------ Before Step: Step {} is starting...", stepExecution.getStepName());
  
              }
  
              @Override
              public ExitStatus afterStep(StepExecution stepExecution) {
                  log.info("------ After Step: Step {} is finished.", stepExecution.getStepName());
                  return stepExecution.getExitStatus();
              }
          };
      }
  }
  ```
- JobExecutionListener와 StepExecutionListener를 등록하여 실행하면
  -  job시작 --> step01 시작 --> step01 종료 --> step02 시작 --> step03 종료 --> job 종료 순으로 수행
 
#### 출처
- https://devocean.sk.com/experts/techBoardDetail.do?ID=167054&boardType=experts&page=&searchData=&subIndex=&idList=&searchText=&techType=&searchDataSub=&searchDataMain=&writerID=kido&comment=
- https://devocean.sk.com/experts/techBoardDetail.do?ID=167161&boardType=experts&page=&searchData=&subIndex=&idList=&searchText=&techType=&searchDataSub=&searchDataMain=&writerID=kido&comment=
