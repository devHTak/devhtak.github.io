---
layout: post
title: File Upload, Download
summary: Spring MVC Basic
author: devhtak
date: '2021-09-17 21:41:00 +0900'
category: Spring
---

#### File Upload

- HTML Form 전송 방식
  - application/x-www-form-urlencoded
    
    <img width="597" alt="image" src="https://user-images.githubusercontent.com/42403023/133733150-dc06ae4d-cbc9-4733-91da-76167f5b6a93.png">
    
    - 이미지 출처: 스프링 MVC 2편 - 백엔드 웹 개바 활용 기술 \[인프러 김영한님 강의]
    - HTML 폼데이터를 전송할 때 사용하는 기본적인 방법
    - HTTP Body에 문자로 username=kim&age=20와 같이 &로 구분하며 key, value를 나타낸다.
    
  - multipart/form-data
    <img width="586" alt="image" src="https://user-images.githubusercontent.com/42403023/133733432-e1d8f0fd-f428-4c64-a3d9-98f754a08c45.png">
    
    - 이미지 출처: 스프링 MVC 2편 - 백엔드 웹 개바 활용 기술 \[인프러 김영한님 강의]
    - 문자와 바이너리를 전송해야 하는 상황에 사용
    - 전송하는 데이터를 Part로 나누어 전송 (구분자: boundary)

#### Servlet File Upload

- MultipartHttpServletRequest와 MultipartResolver
  ```java
  @PostMapping("/servlet/upload")
	public String fildUpload(HttpServletRequest request) throws ServletException, IOException {
		log.info("request: {}", request);

		String itemName = request.getParameter("itemName");
		log.info("itemName: {}", itemName);

		Collection<Part> parts = request.getParts();
		log.info("parts: {}", parts);

		return "ok";
	}
  ```
  - request.getParts(): multipart/form-data 전송 방식에 각각 나누어진 부분을 받아서 확인할 수 있다.
    ```
    Content-Type: multipart/form-data; boundary=----xxxx
    ------xxxx
    Content-Disposition: form-data; name="itemName"

    Spring
    ------xxxx
    Content-Disposition: form-data; name="file"; filename="test.data"
    Content-Type: application/octet-stream
    sdklajkljdf...
    ```
    
- Multipart 사용 옵션
  - 업로드 사이즈 제한
    ```
    spring.servlet.multipart.max-file-size= 1MB
    spring.servlet.multipart.max-request-size= 10MB
    ```
    - Size를 넘어서면 예외(SizeLimitExceeededException) 발생
  
  - multipart enable 끄기
    ```
    spring.servlet.multipart.enabled=false
    ```
    - request.getParts() 의 결과가 비어있다
    - request 객체
      - multipart 사용 X: org.apache.catalina.connector.RequstFacade@xxx
      - multipart 사용 O: org.springframework.web.multipart.support.StandartMultipartHttpServletRequest
    - Multipart를 사용하면 DispatcherServlet엣 멀티파트 리졸버(MultipartResolver)를 실행
    - 일반적인 HttpServletRequest에서 MultipartHttpServletRequest로 변환하여 반환
    - 스프링이 제공하는 기본 멀티파트 리졸버는 MultipartHttpServletRequest 인터페이스를 구현한StandardMultipartHttpServletRequest

- 파일 업로드 예시
  - application.properties
    ```
    file.dir=/Users/htak/study/file/
    ```
  - Controller
    ```java
    @Value("${file.dir}")
    private String fileDir;
    
    @PostMapping("/servlet/upload")
    public String fildUpload(HttpServletRequest request) throws ServletException, IOException {
      log.info("request: {}", request);

      String itemName = request.getParameter("itemName");
      log.info("itemName: {}", itemName);

      Collection<Part> parts = request.getParts();
      log.info("parts: {}", parts);
      for (Part part : parts) {
        log.info("part: {}", part);
        for (String headerName : part.getHeaderNames()) {
          log.info("headerName: {}, headerValue: {}", headerName, part.getHeader(headerName));
        }

        log.info("submittedFileName={}", part.getSubmittedFileName());
        log.info("size={}", part.getSize());

        InputStream inputStream = part.getInputStream();
        String body = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("body: {}", body);

        if(!StringUtils.hasText(body)) {
          String fullPath = filePath + part.getSubmittedFileName();
          log.info("FilePath: {}", filePath);
          part.write(fullPath);
        }
      }
      return "ok";
    }
    ```
    - part.getSubmittedFileName() : 클라이언트가 전달한 파일명
    - part.getInputStream(): Part의 전송 데이터를 읽을 수 있다.
    - part.write(...): Part를 통해 전송된 데이터를 저장할 수 있다.

#### Spring File Upload

- Spring은 MultipartFile이라는 인터페이스로 Multipart File을 매우 편리학 지원한다.
- 예제
  ```java
  @PostMapping("/spring/upload")
	public String springUpload(@RequestParam String itemName, @RequestParam MultipartFile file, HttpServletRequest request) throws IOException {
		log.info("request={}", request);
		log.info("itemName={}", itemName);
		log.info("multipartFile={}", file);

		if(!file.isEmpty()){
			String fullPath = filePath + file.getOriginalFilename();
			log.info("File Path: {}", fullPath);
			file.transferTo(new File(fullPath));
		}

		return "upload-form";
	}
  ```
  - @RequestParam MultipartFile file
    - 업로드하는 HTML Form의 name에 맞추어 @RequestParam 을 적용하면 된다. 추가로 @ModelAttribute 에서도 MultipartFile 을 동일하게 사용할 수 있다.
  - MultipartFile 주요 메서드
    - file.getOriginalFilename() : 업로드 파일 명
    - file.transferTo(...) : 파일 저장


#### 출처

- 스프링 MVC 2편 - 백엔드 웹 개바 활용 기술 \[인프러 김영한님 강의]
