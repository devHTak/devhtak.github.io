---
layout: post
title: 비동기 RestTemplate과 비동기 MVC/Servlet
summary: Reactive Programming
author: devhtak
date: '2021-08-28 21:41:00 +0900'
category: Reactive
---

#### ThreadPool Hell

- 아주 빠르게 무언가를 계산하고 해당 처리를 끝내는 경우라면 굳이 비동기 MVC(서블릿)를 사용하지 않아도 문제가 없다.
- 하지만 분산 환경에서 여러 서비스에 대한 호출을 하거나 외부 서비스에 호출을 하는 경우가 빈번해지면서 비동기로 처리하기 부분이 존재하게 된다.
  - NIO Connector를 사용하여 요청에 대한 Thread는 바로 반납이 가능하지만 Worker Thread에 경우 계속해서 생겨나야 한다.
  - 비동기 서블릿을 사용하더라도 하나의 요청을 처리하는 동안 하나의 작업(워커) 스레드는 그 시간동안 대기상태에 빠지게 되어 결국에는 스레드 풀의 가용성이 떨어지게 된다.
- Thread Pool Hell이란 풀 안에 있는 스레드에 대한 사용 요청이 급격하게 증가해 추가적인 요청이 들어올 때, 사용 가능한 스레드 풀의 스레드가 없기 때문에 대기 상태에 빠져 요청에 대한 응답이 느려지게 되는 상태

- 예시
  - localhost:8080/rest/hello 를 호출하면 핸들러 내에서 localhost:8081/anoter-service를 호출하여 리턴한다.
  - localhost:8080/rest/hello
    ```java
    RestTemplate restTemplate = new RestTemplate();
    @GetMapping("/rest/hello")
    public String hello(@RequestParam int index) {
    	String returnValue = restTemplate.getForObject("http://localhost:8081/another-service?req=${req}", String.class, "hello " + index);
    	return returnValue;
    }
    ```
    - 톰캣에서 생성하는 Thread는 1개로 설정하였다.
      ```
      server.tomcat.threads.max=1
      ```
      
  - localhost:8081/another-service
    - 한 클래스 내에 새로운 서비스를 생성하기 위해서 @SpringBootApplication을 사용하였고, property(port, max-thread) 를 따로 생성하였다.
    - 해당 클래스를 따로 spring boot application을 run 하면 된다.
    ```java
    @SpringBootApplication
    public class AnotherService {	
	    @RestController
	    public static class MyController {
		    @GetMapping("/another-service")
		    public String anotherService(@RequestParam String req) {
			    return "another-service " + req;
		    }
	    }
	    public static void main(String[] args) {
		    System.setProperty("server.port", "8081");
		    System.setProperty("server.tomcat.threads.max", "1");
		    SpringApplication.run(AnotherService.class, args);
	    }
    }
    ```

  - Load Test
    ```java
    public static void main(String[] args) throws InterruptedException {
	// 100개 쓰레드 생성
    	ExecutorService executorService = Executors.newFixedThreadPool(100);	
    	RestTemplate restTemplate = new RestTemplate();
    	String url = "http://localhost:8080/rest/hello?index={index}";
    	CyclicBarrier barrier = new CyclicBarrier(100); // 동기화
    	StopWatch mainStopWatch = new StopWatch();
    	mainStopWatch.start();
    	for(int i = 0; i < 100; i++) {
    		executorService.submit(() -> {
    			int index = counter.addAndGet(1);
    			barrier.await(); // 생성 당시 정해놓은 partition까지 blocking을 생성한다. 
    			StopWatch subStopWatch = new StopWatch();
    			log.info("Thread {}", index);
    			subStopWatch.start();
    			String returnValue = restTemplate.getForObject(url, String.class, index);
    			subStopWatch.stop();
    			log.info("Elapsed: {}, {} / {}", index, subStopWatch.getTotalTimeSeconds(), returnValue);
    			return "good";
    		});
    	}	
    	executorService.shutdown();
    	executorService.awaitTermination(100, TimeUnit.SECONDS);
    	mainStopWatch.stop();
    	log.info("Terminated: {}", mainStopWatch.getTotalTimeSeconds());
    }
    ```
    - CyclicBarrier를 통해서 동기화를 만들었다.
      - 동작 방식은 await()을 만나면 생성자에서 설정한 partition개수 만큼에 Thread를 대기한 후, 실행한다.
  - 간단한 예시지만 hello 핸들러가 처리하는 Worker Thread 내부에서는 anoter-service를 호출하는 과정에서 blocking 되기 때문에 CPU는 놀고 있지만 요청을 빠르게 처리하지 못하게 된다.
  

#### AsyncRestTemplate

- API를 호출하는 작업을 비동기적으로 처리하는 방법으로 AsyncRestTemplate은 RestTemplate을 비동기로 지원하며 Non-blocking이다.
  ```java
  AsyncRestTemplate asyncRestTemplate = new AsyncRestTemplate();
  @GetMapping("/rest/hello")
  public ListenableFuture<ResponseEntity<String>> hello(@RequestParam int index) {
  	return asyncRestTemplate.getForEntity("http://localhost:8081/another-service?req=${req}", String.class, "hello " + index);
  }
  ```
    - ListenableFuture을 바로 리턴이 가능하다.
  - tomcat 스레드는 요청에 대한 작업을 다 끝내기 전에 반환을 해서 바로 다음 요청을 처리하도록 사용
  - 외부 서비스로부터 실제 결과를 받고 클라이언트의 요청에 응답을 보내기 위해서는 새로운 스레드를 할당 받아 사용
  - 외부 서비스로부터 실제 결과를 받고 클라이언트에 응답을 보내기 위해서는 새로운 스레드를 할당 받아야 하지만, 외부 API를 호출하는 동안은 스레드(tomcat) 자원을 낭비하고 싶지 않다는 것이 목적
  
- Spring Boot 2.0 부터 Deprecated되었으며 WebClient를 사용해야 한다.

#### DeferredResult

- DeferredResult를 사용
  - AsyncRestTemplate으로 가져온 결과는 ListenableFuture이다.
  - callback을 활용하여 DeferredResult에 결과를 저장하여 사용할 수 있다.
  ```java
  AsyncRestTemplate asyncRestTemplate = new AsyncRestTemplate();
  @GetMapping("/rest/hello")
  public DeferredResult<String> hello(@RequestParam int index) {
  	DeferredResult<String> result = new DeferredResult<>();
  	ListenableFuture<ResponseEntity<String>> future = asyncRestTemplate.getForEntity("http://localhost:8081/another-service?req=${req}", String.class, "hello " + index);
  	future.addCallback(s-> {
  		result.setResult(s.getBody());
  	}, e -> {
  		result.setErrorResult(e.getMessage());
  	});
  	return result;
  }
  ```
  
#### Callback Hell

- AsyncRestTemplate을 활용하여 여러 서비스를 호출한다면, Callback을 계속 걸어주어야 하는 현상이 발생한다.
  ```java
  AsyncRestTemplate asyncRestTemplate = new AsyncRestTemplate();
  @GetMapping("/rest/hello")
  public DeferredResult<String> hello(@RequestParam int index) {
  	DeferredResult<String> result = new DeferredResult<>();
  	ListenableFuture<ResponseEntity<String>> future1 = asyncRestTemplate.getForEntity("http://localhost:8081/another-service?req=${req}", String.class, "hello " + index);
  	future1.addCallback(s-> {
  		result.setResult(s.getBody());
		ListenableFuture<ResponseEntity<String>> future2 = asyncRestTemplate.getForEntity("http://localhost:8081/another-service2", String.class);
		future2.addCallback(s2 -> {
			result.setResult(s2.getBody());
		}, e2 -> {
			result.setErrorResult(e2.getMessage());
		}); 
  	}, e -> {
  		result.setErrorResult(e.getMessage());
  	});	
  	return result;
  }
  ```
  - 계속해서 service를 호출하게 되면 callback 안으로 들어가야 한다.

#### 출처

- 토비의 봄 TV 9회 스프링 리액티프 프로그래밍(5) - 비동기 RestTemplate과 비동기 MVC/Servlet
