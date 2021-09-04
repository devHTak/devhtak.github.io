---
layout: post
title: WebFlux
summary: Reactive Programming
author: devhtak
date: '2021-08-30 21:41:00 +0900'
category: Reactive
---

#### WebClient

- Spring 5 에서 Non-blocking 방식을 지원하기 위해 제공되었다.
- 기존 AsyncRestTemplate과 콜백 헬을 벗어나기 위한 CompletableFuture와 다르게 편리하게 구현이 가능하다.
- 요청자를 Consumer, 제공자를 Producer 로 요청자의 request에 대해 제공자가 API를 제공할 때 HttpClient를 많이 사용한다.
  - WebClient 도 HttpClient에 일종
  
- RestTemplate vs WebClient
  
  - 공통점: 둘다 HttpClient모듈
  - 차이점: 통신방법이 RestTemplate은 Blocking방식이고, WebClient는 Non-Blocking방식
  - RestTemplate
    - Multi-Thread + Blocking 방식
    
    ![image](https://user-images.githubusercontent.com/42403023/132087051-21833858-4565-4d78-9cf0-62d39c40e4ec.png)
    
    - 이미지 출처: https://happycloud-lee.tistory.com/220
    - 요청 당 Thread를 할당하여 사용
      - 사용 가능한 Thread가 없는 경우 Queue에서 요청은 대기하게 된다.

  - WebClient
    - Single Thread + Non-Blocking 방식
    - Client의 Request는 Event Loop에 등록하고, 이에 대한 결과를 기다리지 않고 다른 작업을 처리한다.
    - Event Loop 는 제공자로부터 callback으로 응답이 오면, 그 결과를 client에게 제공한다.

- Reactive Web
  - Handler의 Return type이 Mono 또는 Flux이다.
  - 요청하는 client가 subscriber이고, 핸들러가 publisher이다.

#### 예시

```java
@RestController
@Slf4j
@RequiredArgsConstructor
public class WebClientExample {
	
	static final String URI1 = "http://localhost:8081/service1?req={req}";
	static final String URI2 = "http://localhost:8081/service2?req={req}";
	
	private final WebClient webClient = WebClient.create();
	private final MyService myService;
	
	@GetMapping("/rest")
	public Mono<String> rest(@RequestParam int index) {
//		Mono<String> body = webClient.get().uri(URI1, index).exchange() // Mono<ClientResponse>
//				.flatMap(clientResponse -> clientResponse.bodyToMono(String.class)) // Mono<String>
//				.flatMap(res1 -> webClient.get().uri(URI2, res1).exchange()) // Mono<ClientResponse>
//				.flatMap(clientResponse -> clientResponse.bodyToMono(String.class)) // Mono<String>
// 				.flatMap(res2 -> Mono.fromCompletionStage(myService.work(res2))); // CompletableFuture<String> -> Mono<String>
		
    // exchange()가 deprecated 되어 다시 작성
		Mono<String> body = webClient.get().uri(URI1, index)
				.exchangeToMono(clientResponse -> clientResponse.bodyToMono(String.class))
				.flatMap(res1 -> webClient.get().uri(URI2, res1)
						.exchangeToMono(clientResponse -> clientResponse.bodyToMono(String.class)))
				.flatMap(res2 -> Mono.fromCompletionStage(myService.work(res2)));  
		
		return body;
	}
	
	@Service
	public static class MyService {
		@Async
		public CompletableFuture<String> work(String req) {
			return CompletableFuture.completedFuture(req + "/asyncValue");
		}
	}
}

@SpringBootApplication
public class AnotherServiceApplication {
	
	@RestController
	public static class anotherController {
		
		@GetMapping("/service1")
		public String service1(@RequestParam String req) {
			return "service1 : " + req;
		}
		
		@GetMapping("/service2")
		public String service2(@RequestParam String req) {
			return "service2 : " + req;
		}
	}
	
	public static void main(String[] args) {
		System.setProperty("server.port", "8081");
		System.setProperty("server.tomcat.threads.max", "1");
		
		SpringApplication.run(AnotherServiceApplication.class, args);
	}

}
```


#### 출처

- 토비의 봄 TV 12회 스프링 리액티프 프로그래밍(8) - WebFlux
- https://happycloud-lee.tistory.com/220
