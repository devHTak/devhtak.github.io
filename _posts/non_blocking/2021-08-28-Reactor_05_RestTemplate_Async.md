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
    
    ```
  
  - 간단한 예시지만 hello 핸들러가 처리하는 Worker Thread 내부에서는 anoter-service를 호출하는 과정에서 blocking 되기 때문에 CPU는 놀고 있지만 요청을 빠르게 처리하지 못하게 된다.

#### AsyncRestTemplate

#### 출처
