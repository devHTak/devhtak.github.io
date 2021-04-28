---
layout: post
title: TDD Patterns
summary: 테스트주도개발 TDD실천법과 도구
author: devhtak
date: '2021-02-21 22:41:00 +0900'
category: Test
---

#### 체크리스트

- 일반적인 애플리케이션
  - 생성자 테스트
  - DTO 객체 생성 테스트
  - 닭과 달걀 메소드 테스트
  - 배열 테스트
  - 객체 동치성 테스트
  - 컬렉션 테스트
- 전통적으로 잘못 인식되어 있는 테스트 메소드의 리팩토링

#### 일반적인 애플리케이션

- TDD가 가장 적극적으로 사용되고 효율이 높은 부분은 애플리케이션의 업무로직을 구현할 때다.
- '로직 구현'이 개발의 중심이 되는 일반적인 개발에서는 테스트 케이스를 작성할 때 어떠한 상황에 접하게 되는지 살펴볼 필요가 있다.

- 생성자 테스트
  - 객체 생성을 위해 반드시 갖춰야 하는 값을 생성자에 설정하는 경우 필요에 따라 테스트를 작성한다.
  - 가끔 생성자 내에 선행 조건, 업무로직을 직접 기술하는 경우도 있는데, 이 때에는 테스트 케이스를 작성해야 한다.

- DTO 스타일 객체 테스트
  - setter/getter로만 이뤄진 DTO 스타일은 따로 테스트 케이스를 작성하지 않는다.
  - 다만 특정한 목적을 갖고 만들어지는 불변 객체(immutable object)의 경우에는 getter 계열 메소드나 상태 확인 is계열의 메소드를 이용한 테스트 케이스를 작성한다.
  
- 닭과 달걀 메소드 테스트
  - 메서드가 서로 맞물려 있어, 완전히 하나만 독립저그올 테스트하기 어려운 경우가 있다.
  - 보통 로직 메소드(add, remove, set)와 상태 확인 메소드(get, show, is)가 짝을 이루는 경우다.
  
  - 예제
  ```java
  void add(String name) {
      attendeeList.add(name);
  }
  
  String get(int order) {
      return attendeeList.get(order);
  }
  ```
  ```java
  @Test
  public void testAdd() throws Exception {
      Attendee attendee = new Attendee();
      attendee.add("TEST");
      assertEquals("TEST", attendee.get(0));
  }
  @Test
  public void testGet() throws Exception {
      Attendee attendee = new Attendee();
      attendee.add("TEST");
      assertEquals("TEST", attendee.get(0));
  }
  ```
  - 해결책 1. 실패하는 테스트 케이스가 두개인 상태에서 작업하기
  - 해결책 2. 안정성이 검증된 제 3의 모듈 사용
  - 해결채 3. 자바 리플렉션을 이용해 강제로 확인하기
    ```java
    @Test
    public void testAddByReflection() throws Exception {
        Attendee attendee = new Attendee();
        attendee.add("TEST");
        
        Field attendeeList = attendee.getClass().getDeclaredField("attendeeList");
        attendeeList.setAccessible(true);
        assertEquals("TEST", (List<String>)attendeeList.get(0));
    }
    ```
    - 리플랙션을 통해 attendeeList를 강제로 소환한다.
    - private 변수지만 강제로 접근이 가능하다.
    - 하지만 리플렉션을 테스트 케이스 작성에 사용하는 일은 부득이한 경우가 아니면 사용하지 않는다.
    
- 배열 테스트
  - assertArrayEquals를 활용한다.

- 객체 동치성 테스트
  - 객체와 객체가 비교할 때는 동일한 객체인지를 판별해야 하는 동일성 테스트, 같은 값인지 확인하는 동치성 테스트인지를 구분해서 생각해야 한다.
  - 필드가 많지 않은 경우 내부 상태(보통은 필드값)를 직접 꺼내와서 각각 비교한다.
  - toString을 중첩구현해 놓고 toString 값을 비교한다.
  - equals 메소드를 중첨구현한다.
    - 가장 이론적이다.
  
- 컬렉션 테스트
  - 기본형이나 String이 컬렉션에 들어있는 겨우
    - assertEquals로 바로 비교 가능하다.
  - 일반 객체가 컬렉션에 들어 있는 경우
    - equals()를 오버라이딩하여 구현하거나 toString()을 오버라이딩하여 구현한다.

#### MVC 아키텍처

- 뷰 TDD
  - HttpUnit
    - HttpUnit은 웹 페이지의 요소들을 테스트하기 위해 사용하는 프레임워크
    - 일반적인 텍스트, 폼이나 링크, 프레임 등에 접근해서 상태를 값으로 불러올 수 있게 한다.
    - 예제
      - 연결 및 확인
      ```java
      @Test
      public void connectTest() throws Exception {
          WebConversion wc = new WebConversion();
          WebResponse response = wc.getResponse("http://www.google.com");
          assertNotNull(response);
          assertEquals("content-type", "text/html", response.getContentType());
          assertThat("메시지 출력 확인", response.getText(), containsString("google"));
      }
      ```
      - 페이지 내의 특정 테이블 확인
      ```java
      WebTable table = response.getTables()[0];
      assertEquals("rows", 4, table.getRowCount());
      assertEquals("columns", 3, table.getColumnCount());
      assertEquals("links", 1, table.getTableCell(0, 2).getLinks().length);
      ```
  - Selenium, CubicTest 등이 있다.
  
- Controller TDD
  - 뷰로부터 넘어오는 요청을 가상으로 만들어주고, 그 결과에 해당하는 응답이 일치하는지 판단하는 것이 가장 간단한 방법이다.
  - MockMvc 사용 이전 test 코드
    ```java
    @Test
    public void testSearchByEmpid() throws Exception {
        MockHttpServletRequest requset = new MockHttpServletRequest();
        MockHttpServletResponse response = new MockHttpServletResponse();
        
        request.addParameter("empid", "5874");
        EmployeeSearchServlet searchServlet = new EmployeeSearchServlet();
        searchServlet.service(request, response);
        
        Employee employee = (Employee)request.getAttribute("employee");
        assertEqluals("박성철", employee.getName());
    }
    ```
  - MockMvc, Mockito 사용
    ```java
    @Autowired MockMvc mockMvc;
    @Autowired SearchService searchService;
    
    @Test
    public void testSearchByEmpidUsingMockMvc() throws Exception {
        Employee expectedEmployee = Employee.builder().name("박상철").id("5874").build();
        when(searchService.findById("5874")).thenReturn(expectedEmployee);
        
        mockMvc.perform(get("/employee/{empId}", expectedEmployee.getId())
            .andDo(print())
            .andExpect(status().isOk());
          
        Employee employee = searchService.findById(expectedEmployee.getId());
        assertEqluals(expectedEmployee.getName(), employee.getName());
    }
    ```
- 모델 TDD
  - 도메인 모델에 대한 TDD
    - 도메인 모델에 경우 단순하기 때문에 작성 필요성이 없을 수 있지만, 테스트 커버리지를 높일 필요가 있는경우 간단하게 작성할 필요도 있다.
    - 만약 상태 변화 등 내부 로직 확인이 필요한 경우 해당 테스트 로직은 작성하자.
  - 서비스 모델에 대한 TDD
    - Web Application에서 가장 중요한 부분을 차지하기 때문에 테스트 커버리지를 높일 필요성이 있다.

#### Anti-Pattern

- 기본적으로 좋은 테스트 케이스는 아래 규칙을 따른다.
  - 하나의 테스트 케이스는 외부와 독립적이어야 한다. 따라서 다른 테스트 케이스에 영향을 주거나 받지 않아야 한다.
  - 하나의 일관된 시나리오를 갖고 있어야 한다.

- @TestInstance(PER_CLASS)
  - 테스트 클래스의 다음과 같이 애노테이션을 붙이면 메소드 단위로 테스트를 실행하는 것이 아닌, 클래스단위로 생성한다.
  - 초기화를 할 때 메소드 단위는 @BeforeEach를 사용하지만 클래스 단위는 @BeforeAll을 사용하면 된다.
  - 메소드 단위를 했을 때 중복이 발생하는데, 각각 독립된 테스트 시나리오에 해당하기 때문에 가독성 면에서 더 좋다.

** 참고 서적: 테스트주도개발 TDD실천법과 도구
