---
layout: post
title: Reactor, 자바와 Spring 비동기 기술
summary: Reactive Programming
author: devhtak
date: '2021-08-09 21:41:00 +0900'
category: Reactive
---

#### java.util.concurrent.Future

- 비동기적인 연산, 작업에 대한 결과를 갖는다.
- 쓰레드가 같은 경우에는 return을 받으면 되지만, 다른 쓰레드에 결과를 받기위한 인터페이스
- Thread Pool
  - Thread를 새로 만드는 것은 리소스가 많이 들기 때문에 Pool에 여러 쓰레드를 생성해 두고, 필요할 때 가져다 쓰고 반납하도록 하여 자원 낭비를 최소화하는 방법
  ```java
  public static void main(String[] args) throws InterruptedException, ExecutionException {
		ExecutorService executorService = Executors.newCachedThreadPool();
		
		Future<String> future = executorService.submit(()->{
			Thread.sleep(200);
			log.info("Async");
			return "Hello";
		});
		
		log.info(future.get());
		log.info("Exit");
	}
  ```
  ```
  11:21:05.286 [pool-1-thread-1] INFO com.example.demo.java.SpringReactorController - Async
  11:21:05.294 [main] INFO com.example.demo.java.SpringReactorController - Hello
  11:21:05.294 [main] INFO com.example.demo.java.SpringReactorController - Exit
  ```
    - ExecutorService의 submit을 통해 Callable 이나 Runnable을 받을 수 있다.
    - Thread는 Callable이나 Runnable의 구현된 메서드를 수행한다는 공통점이 있지만, 아래와 같은 차이점이 있다.
      - interface Callable: V call() throws Exception: 리턴값이 존재하며 Exception을 던질 수 있다.
      - interface Runnable: void run() : 인자, 결과값 리턴이 없다.


#### 출처

- 토비의 봄 TV 8회 스프링 리액티프 프로그래밍(4) - 자바와 스프링의 비동기 기술
