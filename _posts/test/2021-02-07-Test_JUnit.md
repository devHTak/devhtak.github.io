---
layout: post
title: 더 자바 테스트1. JUnit
summary: JUnit 사용법
author: devhtak
date: '2021-02-07 21:41:00 +0900'
category: Test
---

#### JUnit 5

- 자바 개발자가 가장 많이 사용하는 테스팅 프레임워크
- Java 8 이상을 필요로 한다.
- 대체제로 TestNG, Spock 등이 있다.
- 구성
  - Platform: JUnit으로 작성한 테스트 코드를 실행해주는 런처 제공, TestEngine API 제공
  - Jupiter: TestEngine API 구현체로 JUnit5 제공
  - Vintage: JUnit3, 4를 지원하는 TestEnging 구현체

- 참고: https://junit.org/junit5/docs/current/user-guide/

#### JUnit 5 시작하기

- Spring Boot 2.2+ 이상의 프로젝트를 만든다면 기본으로 JUnit5 의존성이 추가된다.
- JUnit4까지는 class, test method가 모두 public 접근지시자야 했지만 JUnit5 부터는 class, test method가 default 접근지시자여도 괜찮다.
- dependency 추가
  ```
  <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>5.5.2</version>
      <scope>test</scope>
  </dependency>
  ```
- 기본 애노테이션
  - @Test
    - 테스트 메소드에 붙여준다.
  - @BeforeAll / @AfterAll
    - static void로 작성해야 한다. 
    - 접근 지시자는 private은 안되고, 나머지는 가능하다.
    - 테스트 코드가 시작되기 전, 후에 한번씩 실행된다.
  - @BeforeEach / @AfterEach
    - 접근 지시자는 private은 안되고, 나머지는 가능하다.
    - 테스트 메소드가 시작하기 전, 후에 각각 실행된다.
  - @Disabled
    - 테스트를 실행하고 싶지 않은 경우에 사용한다.
  
  ```java
  public class StudyTest {	
      @BeforeAll
      static void beforeAll() {
          System.out.println("beforeAll");
      }
      @AfterAll
      static void afterAll() {
          System.out.println("afterAll");
      }
      @BeforeEach
      void beforeEach() {
          System.out.println("beforeEach");
      }
      @AfterEach
      void afterEach() {
          System.out.println("afterEach");
      }
      @Test
      void createInstance() {
          Study study = new Study();
          assertNotNull(study);
      }
      @Test
      void sampleTest() {
          System.out.println("Hello JUnit5");
      }
  }
  ```
  ```
  // 출력 화면
  beforeAll
  beforeEach
  Hello JUnit5
  afterEach
  beforeEach
  afterEach
  afterAll
  ```

#### 테스트 이름 표시하기

- @DisplayNameGeneration
  - Mehtod와 Class 레퍼런스를 사용해서 테스트 이름을 표기하는 방법 설정
  - 기본 구현체로 ReplaceUnderscores 제공
  
- @DisplayName
  - 어떤 테스트인지 테스트 이름을 보다 쉽게 표현할 수 있는 방법을 제공하는 애노테이션
  - @DisplayNameGeneration 보다 우선 순위가 높다.
  
```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
public class StudyTest {
    @Test
    @DisplayName("스터디_만들기")
    void test() {
        System.out.println("test");
    }
}
```
  - @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
    - create_instance를 create instance로 보이게 한다.
  
- 참고: https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names

#### Assertion

- Assertion 메소드
  - 패키지: org.junit.jupiter.api.Assertions.*
  
  |메소드|설명|
  |---|---|
  |assertEquals(expected, actual)|실제 값이 기대한 값과 같은지 확인|
  |assertNotNull(actual)|값이 null이 아닌지 확인|
  |assertTrue(boolean)|다음 조건이 참(true)인지 확인|
  |assertAll(excutables...)|모든 확인 구문 확인|
  |assertThrows(expectedTyupe, excutable)|예외 발생 확인|
  |assertTimeout(duration, excutable)|특정 시간 안에 실행이 완료되는지 확인|
  
  - 파라미터의 순서는 보통은 기대값, 비교하고자 하는 값을 입력한다. 
  
- 마지막 매개변수로 Supplier<String> 타입의 인스턴스를 람다 형태로 제공할 수 있다.
  - 복잡한 메시지 생성해야 하는 경우 사용하면 실패한 경우에만 해당 메시지를 만들게 할 수 있다.
- 다른 다양한 라이브러리를 사용할 수 있다.
  - AssertJ: https://joel-costigliola.github.io/assertj
    ```java
    assertThat(study.getLimit()).isGreaterThan(10);
    ```
  - Hemcrest: https://hamcrest.org/JavaHamcrest/
  - Truth: https://truth.dev/

```java
@Test
void create_instance() {
    Study study = new Study();
    assertNotNull(study);
    assertEquals(StudyStatus.DRAFT, study.getStatus(), new Supplier<String>() {
        @Override
        public String get() {
            // TODO Auto-generated method stub
            return null;
        }			
    }); //lambda 변환 가능 () -> "message";

    assertTrue(study.getLimit() >= 0, () -> "스터디 최소 참가자는 0 이상이어야 합니다.");

    assertAll(
        ()-> assertNotNull(study),
        ()-> assertEquals(StudyStatus.DRAFT, study.getStatus()),
        ()-> assertTrue(study.getLimit() >= 0, () -> "스터디 최소 참가자는 0 이상이어야 합니다.")
    );

    Exception e = assertThrows(IllegalArgumentException.class, ()-> study.setLimit(-10));
    assertEquals("limit은 0 이상이어야 한다.", e.getMessage());

    assertTimeout(Duration.ofMillis(100), ()->{
        new Study();
        Thread.sleep(1000);
    });

    assertTimeoutPreemptively(Duration.ofMillis(100), ()->{
        new Study();
        Thread.sleep(1000);
    });
}
```
- assertAll은 여러 assertion을 람다식으로 구현하여 한번에 테스트할 수 있다.
- assertThrows는 발생한 Exception 을 리턴한다.
- assertTimeout은 excutable이 완료될 때까지 기다린 후에 완료되기 때문에, 시간이 오래 걸린다.
  - assertTimeoutPreemptively(duration, excutable)
    - 정해놓은 Duration만큼만 기다리고 실패하면 바로 끝낸다.
    - 별도의 thread를 만들어 사용한다.
    - ThreadLocal(Spring Transaction 등)을 사용하면 예상치 못한 결과를 얻을 수 있다.
    - 다른 스레드에서 ThreadLocal이 공유되지 않기 때문에 트랜잭션 설정이 제대로 안될 수 있다. 기본 rollback인데 commit이 될 수도 있다.
    - 이 동작은 executable 또는 내부에서 실행되는 코드가 스토리지에 supplier의존 하는 경우 바람직하지 않은 부작용을 초래할 수 있다.

#### 조건에 따라 테스트 실행하기

- 특정한 조건을 만족하는 경우에 테스트를 실행하는 방법.
- org.junit.jupiter.api.Assumptions.*
  - assumeTrue(조건)
    - 조건이 참이면 아래 라인에 테스트를 마저 진행하고, 거짓인 경우에 중단된다.
    - assumeTrue가 실패하면 Assumption failed: assumption is not true 라는 메시지와 함께 테스트가 더이상 진행되지 않는다.
      ```java
      @Test
      @DisplayName("스터디 만들기")
      void create_new_study() {
          String testenv = System.getenv("testenv");
          assumeTrue("LOCAL".equalsIgnoreCase(testenv));

          Study actual = new Study();
          assertNotNull(actual);

          actual.setLimit(10);
          assertEquals(10, actual.getLimit());
      }
      ```
    - 환경변수 실행 방법
      - System.getenv 는 환경변수를 가져올 수 있다.
      - cmd(window): set TEST_ENV="LOCAL"
      - linux: vim ~/.zhsrc 하여 export TEST_ENV=LOCAL 저장
      
  - assuming That(조건, 테스트)
    - 조건이 참이면 lambda로 표현한 테스트를 실행한다.
    - 조건이 틀리더라도 테스트는 성공으로 표현된다.
      ```java
      @Test
      @DisplayName("스터디 만들기")
      void create_new_study() {
          String testenv = System.getenv("testenv");
          // assumeTrue("LOCAL".equalsIgnoreCase(testenv));

          assumingThat("LOCAL".equalsIgnoreCase(testenv), ()->{
            Study actual = new Study();
            assertNotNull(actual);

            actual.setLimit(10);
            assertEquals(10, actual.getLimit());
          });		
      }
      ```
  
- @Enabled__ 와 @Dsiabled__
  - @EnabledOnOS | @DisabledOnOs    
    - 운영체제에 대해 확인하여 조건에 맞게 실행 및 실행하지 않는다.
    - @EnabledOnOs(OS.MAC)
  - @EnabledOnJre | @DisabledOnJre
    - JRE 버전에 대해 확인하여 조건에 맞게 실행 및 실행하지 않는다.
    - @DisabledOnJre(JRE.JAVA_11)
  - @EnabledIfSystemProperty | @DisabledIfSystemProperty
    - 시스템 설정에 대해 확인하여 조건에 맞게 실행 및 실행하지 않는다.
    - @EnabledIfSystemProperty(named="testenv", matches="LOCAL")
  - @EnabledIfEnvironmentVariable | @DisabledIfEnvironmentVariable
    - 환경변수에 대해 확인하여 조건에 맞게 실행 및 실행하지 않는다.
    - @DisabledIfEnvironmentVariable(named="testenv", matches="LOCAL")
  - @EnabledIf | @DisabledIf
    - 조건에 맞으면 실행 및 실행하지 않는다.
  
#### 태깅과 필터링

- 테스트 그룹을 만들고 원하는 테스트 그룹만 테스트를 실행할 수 있는 기능
- @Tag
  - 테스트 메소드에 태그를 추가할 수 있다.
  - 하나의 테스트 메소드에 여러 태그를 사용할 수 있다.
  
- 메이븐에서 테스트 필터링 하는 방법
  ```
  <plugin>
      <artifactId>maven-surefire-plugin</artifactId>
      <configuration>
          <groups>fast|slow</groups>      
      </configuration>
  </plugin>
  ```

- 예제) fast로 태그한 것은 로컬환경에서 실행되고, slow로 태그한 것은 CI 후 배포 환경에서 실행되도록 하자
  - 테스트 태깅
    ```java
    @Test
    @DisplayName("스터디 만들기")
    @Tag("fast")
    void create_new_study() {
        Study actual = new Study();
        assertNotNull(actual);

        actual.setLimit(10);
        assertEquals(10, actual.getLimit());
    }

    @Test
    @DisplayName("스터디 만들기")
    @Tag("slow")
    void test_timeout() {
        assertTimeout(Duration.ofMillis(1000), ()->{
            new Study();
            Thread.sleep(10000);
        });
    }
    ```
  
  - 메이븐 설정
    ```
    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <groups>fast</groups>      
                    </configuration>
                </plugin>
            </build>
        </profile>
        <profile>
            <id>ci</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                </plugin>
            </build>
        </profile>
    </profiles>
    ```
    ```
    $ mvn test -P ci // profile에서 id가 ci인 환경을 참조하여 모든 테스트를 실행한다.
    $ mvn test // profile에서 id가 default인 환경을  참조하여 fast로 태깅된 테스트만 실행한다.
    ```
  
- 참고
  - https://maven.apache.org/guides/introduction/introduction-to-profiles.html
  - https://junit.org/junit5/docs/current/user-guide/#running-tests-tag-expressions

#### 커스텀 태그

- jUnit5 애노테이션을 조합하여 커스텀 태그를 만들 수 있다.
- 커스텀 태그를 만들어 메타 애노테이션, Composed Annotation로 사용할 수 있다.
  - 즉, 커스텀 태그에 기존 사용하던 애노테이션을 함께 사용하여 같은 기능을 하도록 만들 수 있다.
- 예제) fast, slow를 태깅한 애노테이션 만들기
  - FastTest.java
    ```java
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Tag("fast")
    @Test
    public @interface FastTest {}
    ```
  - SlowTest.java
    ```java
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Tag("slow")
    @Test
    public @interface SlowTest {}
    ```
  
  - 활용하기
    ```java
    @FastTest
    @DisplayName("스터디 만들기 fast")
    void create_new_study() {
        // ...
    }
    
    @SlowTest
    @DisplayName("스터디 만들기 slow")
    void create_new_study_again() {
        // ...
    }    
    ```
#### 테스트 반복하기

- 테스트를 실행할 때마다 반복적으로 랜던값을 만들거나 타이밍에 따라 조건이 있는 경우 반복을 사용할 수 있다.
- @RepeatedTest
  - 반복 횟수와 반복 테스트 이름을 설정할 수 있다.
  - 애노테이션 파라미터로 value에는 반복할 값을 입력, name에는 반복되는 테스트에 이름을 설정할 수 있다.
  - name에는 아래와 같은 변수를 사용할 수 있다.
    - {displayName} : 반복 테스트에서 DisplayName 애노테이션으로 작성한 이름을 사용할 수 있다.
    - {currentRepetition} : 현재 반복 횟수를 사용할 수 있다.
    - {totalRepetitions} : 전체 반복 횟수를 사용할 수 있다.
  - RepetitionInfo 타입의 인자를 받을 수 있다.
    ```java
    @DisplayName("반복 테스트")
    @RepeatedTest(value = 10, name = "{displayName}, {currentRepetition}/{totalRepetitions}")
    void repeatTest(RepetitionInfo repetitionInfo) {
        System.out.println("test " + repetitionInfo.getCurrentRepetition() + " / "
            + repetitionInfo.getTotalRepetitions());
    }
    ```
  
- @ParameterizedTest
  - 테스트에 여러 다른 매개변수를 대입해가며 반복을 실행한다.
  - 애노테이션 파라미터로 name을 설정할 수 있는데 아래와 같은 변수를 사용할 수 있다.
    - {displayName} : @DisplayName 애노테이션에 설정해준 이름 사용
    - {index} : 반복하는 index 사용
    - {arguments} : 사용하는 arguments를 아래와 같은 방법으로 인덱스로 접근할 수 있다.
      - {0}, {1}, ...
    
  - ValueSource로 입력받은 strings 배열을 반복하며 테스트할 수 있다.
    ```java
    @DisplayName("파라미터 테스트")
    @ParameterizedTest(name = "{index} {displayName} message={0}") // 1 파라미터 테스트 message=날씨가
    @ValueSource(strings = {"날씨가", "많이", "추어지고", "있네요."})
    void parameterizedTest(String message) {
        System.out.println(message);
    }
    ```
    
- 인자 값들의 소스 
  - @ValueSource
    - 다양한 타입의 배열로 값을 넘겨주어 반복하여 테스트 파라미터로 넘겨줄 수 있다. 
  - @NullSource, @EmptySource, @NullAndEmptySource
    - null, 비어있는 문자열을 넣어준다.
    - @NullAndEmptySource는 @NullSource와 @EmptySource의 Composite Annotation
  - @EnumSource
  - @MethodSource
  - @CvsSource
  - @CvsFileSource
  - @ArgumentSource
  
- 인자 값 타입 변환
  - 암묵적인 타입 변환
    - 레퍼런스 참고
    - https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit
  - 명시적인 타입 변환
    - SimpleArgumentConverter 상속 받은 구현체 제공    
    - @ConverterWith
      - 인자로 사용할 SimpleArgumentConverter 상속 객체를 지정해준다.
      - 예제에서는 Class 내에 static class로 생성하였다.
      
      ```java
      static class StudyConverter extends SimpleArgumentConverter {
          @Override
          protected Object convert(Object source, Class<?> targetType) throws ArgumentConversionException {
              // TODO Auto-generated method stub
              assertEquals(Study.class, targetType, "Can only convert to Study");
              Study study = new Study();
              study.setLimit(Integer.parseInt(source.toString()));
              return study;
          }		
      }
      ```
      ```java
      @DisplayName("파라미터 테스트")
      @ParameterizedTest(name = "{index} {displayName} message={0}")
      @CsvSource({"10", "20"})
      void parameterizedTest(@ConvertWith(StudyConverter.class) Study study) {
          System.out.println(study.getLimit());
      }
      ```
      
      
    
- 인자 값 조합
  - 예제, 기존 방식
    ```java
    @DisplayName("파라미터 테스트2")
    @ParameterizedTest(name="{index}. {displayName} message={0}, {1}")
    @CsvSource({"10, JAVA", "20, JPA"})
    void parameterizedTest2(Integer limit, String name) {
        Study study = new Study();
        study.setLimit(limit); study.setName(name);
        System.out.println(study.toString());
    }
    ```
  - ArgumentAccessor
    ```java
    @DisplayName("파라미터 테스트2")
    @ParameterizedTest(name="{index}. {displayName} message={0}, {1}")
    @CsvSource({"10, JAVA", "20, JPA"})
    void parameterizedTest2(ArgumentsAccessor argumentsAccessor) {
        Study study = new Study();
        study.setLimit(argumentsAccessor.getInteger(0)); 
        study.setName(argumentsAccessor.getString(1));
        System.out.println(study.getLimit() + " " + study.getName());
    }
    ```
    
  - 커스텀 Accessor
    - ArgumentsAggregator 인터페이스 구현
    - @AggregateWith
    
      ```java
      static class StudyAggregator implements ArgumentsAggregator {
          @Override
          public Object aggregateArguments(ArgumentsAccessor accessor, ParameterContext context)
              throws ArgumentsAggregationException {
              // TODO Auto-generated method stub
              Study study = new Study();
              study.setLimit(accessor.getInteger(0)); 
              study.setName(accessor.getString(1)); 
              return study;
          }
      }
      ```
      ```java
      @DisplayName("파라미터 테스트2")
      @ParameterizedTest(name="{index}. {displayName} message={0}, {1}")
      @CsvSource({"10, JAVA", "20, JPA"})
      void parameterizedTest2(@AggregateWith(StudyAggregator.class) Study study) {
          System.out.println(study.getLimit() + " " + study.getName());
      }
      ```
- SimpleArgumentConverter, ArgumentsAggregator 구현체는 Test 소스내에 Inner class로 작성해야 한다.

- 참고
  - https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests
  
#### 테스트 인스턴스

- JUnit 테스트 메소드마다 테스트 인스턴스를 새로 만든다.
  - 기본 전략
  - 테스트 간에 의존성을 없애고 테스트 메소드를 독립적으로 실행하여 예상치 못한 부작용을 방지하기 위함이다.
  - 이 전략을 JUnit 5에서 변경할 수 있다.
  
- @TestInstance(Lifecycle.PER_CLASS)
  - 테스트 클래스당 인스턴스를 하나만 만들어 사용
  - 경우에 따라, 테스트 간에 공유하는 모든 상태를 @BeforeEach 또는 @AfterEach에서 초기화할 필요가 있다.
  - @BeforeAll과 @AfterAll을 인스턴스 메소드 또는 인터페이스에 정의한 default 메소드로 정의할 수도 있다.
    - static 메소드로 설정해야 한다.
  - 아래와 같이 생성하면 동일한 Study 객체를 모두 사용할 수 있다.
  
    ```java
    @TestInstance(Lifecycle.PER_CLASS)
    public class StudyTest {
        private static Study study;
        @BeforeAll
        static void beforeAll() {
            study = new Study();
        }
    }
    ```

#### 테스트 순서

- 실행할 테스트 메소드 특정한 순서에 의해 실행되지만 어떻게 그 순서를 정하는지는 의도적으로 분명하지 않다.
- 테스트 인스턴스를 테스트마다 새로 만드는 것과 같은 이유
- 경우에 따라, 특정 순서대로 테스트를 실행하고 싶을 때도 있다.
- 테스트 메소드 원하는 순서 지정하기
  - @TestInstance(Lifecycle.PER_CLASS)와 함께 @TestMethodOrder를 사용할 수 있다.
  - MethodOrder 구현체를 설정한다.
  - 기본 구현체
    - Alphanumeric
    - OrderAnnotation
    - Random
    
- OrderAnnotation을 상요한 예제로 Order의 int값이 낮을 수록 빠르게 실행한다.

  ```java
  @TestInstance(Lifecycle.PER_CLASS)
  @TestMethodOrder(MethodOrderer.OrderAnnotation.class)
  public class StudyTest {
      @Test
      @Order(1)
      void create_instance() {...}
      @Test
      @Order(2)
      void getStudyTest() {...}
  }
  ```

#### junit-platform.properties

- JUnit 설정 파일로, 클래스패스 루트(src/test/resources/)에 넣어두면 적용된다.
  - eclipse에서 등록하기
    - src/test 밑에 java폴더만 있기 때문에 resources 폴더를 추가한다.
    - 프로젝트 우클릭 -> properties -> Java Build Path로 이동하여 Add Folder로 resources 폴더를 선택한다.
    - 클릭하여 폴더 하단에 Output 폴더와 Contains test sources를 YES로 바꿔준다.
    - junit-platform.proerties 파일을 만들어 설정 값을 작성한다.
  
- 테스트 인스턴스 라이프사이클 설정
  ```
  junit.jupiter.testinstance.lifecycle.default=per_class
  ```
- 확장 팩 자동 감지 기능
  ```
  junit.jupiter.extensions.autodetection.enabled=true
  ```
- @Disabled 무시하고 실행하기
  ```
  junit.jupiter.extensions.deactive=org.junit.*DisabledCondition
  ```
- 테스트 이름 표기 전략 설정
  ```
  junit.jupiter.displayname.generator.default=\org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores 
  ```
  
#### Extension Model

- JUnit4의 확장 모델은 @RunWith(Runner), TestRule, MethodRult
- JUnit5의 확장 모델은 Extension
- Extension Model 등록 방법
  - 선언적 등록: @ExtendWith
  - 프로그래밍 등록: @RegisterExtension
  - 자동 등록 자바 ServiceLoader 이용
  
- Extension Model 만드는 방법
  - 테스트 실행 조건
  - 테스트 인스턴스 팩토리
  - 테스트 인스턴스 후.. 처리기
  - 테스트 매개변수 리졸버
  - 테스트 라이프사이클 콜백
  - 예외처리
  - ...
  
- 예제, 오래걸리는 테스트 메소드를 찾아내는 Extension Model 만들기
  - 확장 만들기
    ```java
    public class FastSlowTestExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {
        private static final long THRESHOLD = 1000L;
        @Override
        public void afterTestExecution(ExtensionContext context) throws Exception {
            Method method = context.getRequiredTestMethod();
            SlowTest annotation = method.getAnnotation(SlowTest.class);

            ExtensionContext.Store store = this.getStore(context);
            long startTime = store.remove("START_TIME", long.class);
            long duration = System.currentTimeMillis() - startTime;

            if(duration > THRESHOLD && annotation == null) {
                String testMethodName = context.getRequiredTestMethod().getName();
                System.out.printf("Please consider mark method [%s] with @SlowTest.\n", testMethodName);
            }
        }

        @Override
        public void beforeTestExecution(ExtensionContext context) throws Exception {
            // TODO Auto-generated method stub
            ExtensionContext.Store store = this.getStore(context);
            store.put("START_TIME", System.currentTimeMillis());
        }

        private ExtensionContext.Store getStore(ExtensionContext context) {
            String testClassName = context.getRequiredTestClass().getName();
            String testMethodName = context.getRequiredTestMethod().getName();

            return context.getStore(ExtensionContext.Namespace.create(testClassName, testMethodName));
        }
    }
    ```
    - BeforeTestExcutionCallBack 인터페이스의 beforeTestExcution 메소드는 테스트 실행하기 전에 실행되어있다.
    - AfterTestExcutionCallBack 인터페이스의 afterTestExcution 메소드는 테스트 실행하기 전에 실행되어있다.
    - ExtensionContext.Store 에 필요한 데이터를 key, value 형태로 사용할 수 있으며, 이 때 Namespace를 통해 테스트하고 있는 클래스와 메소드 별로 설정할 수 있다.
  
  - Extension Model 등록
    - 선언적 등록: @ExtendWith
      - FastSlowTestExtension 클래스 객체를 생성하여 사용할 수는 없다.
      ```java
      @ExtendWith(FastSlowTestExtension.class)
      public class StudyTest { ... }
      ```
    - 프로그래밍 등록: @RegisterExtension
      - static 메소드에 @RegisterExtension으로 등록하여 사용할 수 있다.
      ```java
      static FastSlowTestExtension fastSlowTestExtension = new FastSlowTestExtension();
      ```
    - 자동 등록 자바 ServiceLoader 이용
      - 자동으로 등록하는 설정은 기본적으로 꺼져있다.
      - 원치않는 Extension Model이 추가될 수도 있기 때문에 조심히 사용해야 한다.
      - junit 공식 문서 5.2.3 Automatic Extension Registration 참조
  
- 참고
  - https://junit.org/junit5/docs/current/user-guide/#extensions
  
#### jUnit4 마이그레이션

- junit-vintage-engine을 의존성으로 추가하면, JUnit5의 junit-platform으로 JUnit 3, 4로 작성된 테스트를 실행할 수 있다.
  - junit-vintage-engine이 exclusion되어 있다면 삭제해주어야 한다.
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependnecy>
    ``` 
  - spring-boot-starter-test에 의존성이 빠져있다면 직접 넣어주어야 한다.
    ```
    <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
    </dependency>
    ```
  
  - @Rule은 기본적으로 지원하지 않지만, junit-jupiter-migratoinsupport 모듈이 제공하는 @EnableRuleMigrationSupport를 사용하면 다음 타입의 Rule을 지원한다.
    - @ExternalResource
    - @Verifier
    - @ExpectedExcpetion
  
  |JUnit4|JUnit5|
  |---|---|
  |@Category(class)|@Tag(class)|
  |@RunWith, @Rule, @ClassRule|@ExtendedWith, @RegisterExtension|
  |@Ignore|@Diabled|
  |@Before, @After, @BeforeClass, @AfterClass|@BeforeEach, @AfterEach, @BaforeAll, @AfterAll|

- 예제
  - import 되어 있는 패키지가 org.junit.jupiter가 아닌 org.junit으로 되어 있다.
  - JUnit4의 애노테이션 등을 사용할 수 있지만, 실제 돌아가는 엔진은 JUnit5이다.
  ```java
  import static org.junit.Assert.assertNotNull;
  import org.junit.After;
  import org.junit.Before;
  import org.junit.Test;

  public class StudyJUnit4Test {	
    @Before
    public void before() {
      System.out.println("before");		
    }

    @After
    public void after() {
      System.out.println("after");
    }

    @Test
    public void createTest() {
      Study study = new Study();
      assertNotNull(study);
    }
  }
  ```


** 참고: 백기선님 인프런 강의 중 더 자바 애플리케이션을 테스트하는 다양한 방법 
