---
layout: post
title: 더 자바 테스트2. Mockito
summary: 더 자바, 애플리케이션을 테스트하는 다양한 방법 2
author: devhtak
date: '2021-02-11 22:41:00 +0900'
category: Test
---

#### Mockito

- Mock: 진짜 객체와 비슷하게 동작하지만, 프로그래머가 직접 그 객체의 행동을 관리하는 객체
- Mockito: Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공하는 프레임워크
  - https://site.mockito.org/
- 테스트를 작성하는 자바 개발자 50% 이상이 사용하는 Mock Framework
  - https://www.jetbrains.com/lp/devecosystem-2019/java/
  - 대체제는 EasyMock, JMock 등이 있다.
- 마틴 파울러의 단위테스트에 대한 고찰
  - https://martinfowler.com/bliki/UnitTest.html
  
#### Mockito 시작하기

- 스프링 부트 2.2+ 프로젝트 생성시 spring-boot-starter-test에 자동으로 Mockito를 추가해준다.
- 스프링 부트를 사용하지 않는다면, 의존성을 직접 추가해야 한다.
  ```
  <!-- mockito  -->
  <dependency>
      <groupId>org.mockito</groupId>
      <artifactid>mockito-core</artifactId>
      <version>3.1.0</version>
      <scope>test</scope>
  </dependency>
  <!-- junit과 mockito를 연결하여 사용할 수 있도록 해준다. -->
  <dependency>
      <groupid>org.mockito</groupId>
      <artifactId>mockito-junit-jupiter</artifactId>
      <scope>test</scope>
  </dependency>
  ```

- 아래 사항만 알면 Mock을 활용한 테스트를 쉽게 작성할 수 있다.
  - Mock을 만드는 방법
  - Mock이 어떻게 동작해야 하는지 관리하는 방법
  - Mock의 행동을 검증하는 방법
  
- Mockito 레퍼런스
  - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html
  
#### Mock 객체 만들기

- Mockito.mock() 메소드로 만드는 방법
  ```java
  MemberService memberService = mock(MemberService.class);
  StudyRepository studyRepository = mock(StudyRepository.class);
  ```

- @Mock 애노테이션으로 만드는 방법
  - JUnit5 extension으로 MockitoExtension을 사용해야 한다.
  - 필드
  - 메소드 매개변수
  ```java
  @ExtendWith(MockitoExtension.class)
  class StudyServiceTest {
      @Mock
      MemberService memberService;
      
      @Mock
      StudyRepository studyRepository;
  }
  ```
  
  ```java
  @ExtendWith(MocktoExtension.class)
  class StudyServiceTest {
      @Test
      void createStudyService(@Mock MemberService memberService, @Mock StudyRepository studyRepository) {
          StudyService studyService = new StudyService(memberService, studyRepository);
          assertNotNull(studyService);
      }
  }
  ```

- 예제
  - MemberService와 StudyRepository interface를 구현하지 않은 체 Mock객체로 생성하여 테스트
    - Member.class와 Study.class
      ```java
      @Entity
      public class Member {
          @Id
          private Long id;

          private String name;

          @OneToMany(mappedBy = "owner")
          private Set<Study> studies = new HashSet<>();

          public Long getId() {
              return id;
          }

          public void setId(Long id) {
              this.id = id;
          }

          public String getName() {
              return name;
          }

          public void setName(String name) {
              this.name = name;
          }

          public Set<Study> getStudies() {
              return studies;
          }

          public void setStudies(Set<Study> studies) {
              this.studies = studies;
          }
      }
      ```
      ```java
      @Entity
      public class Study {
          @Id
          private Long id;
          @ManyToOne
          private Member owner;

          public Long getId() {
              return id;
          }
          public void setId(Long id) {
              this.id = id;
          }
          public Member getOwner() {
              return owner;
          }
          public void setOwner(Member owner) {
              this.owner = owner;
          }
      }
      ```
    - StudyService, StudyRepository 생성
      ```java
      public class StudyService {	
          private final MemberService memberService;
          private final StudyRepository repository;
          public StudyService(MemberService memberService, StudyRepository repository) {
              assert memberService != null;
              assert repository != null;
              this.memberService = memberService;
              this.repository = repository;
          }
          public Study createNewStudy(Long memberId, Study study) {
              Optional<Member> member = memberService.findById(memberId);
              study.setOwner(member.orElseThrow(() -> {
                  throw new IllegalArgumentException("Member doesn't exist for id: " + memberId);
              }));
              return repository.save(study);
          }
      }
      ```
      ```java
      public interface StudyRepository extends JpaRepository<Study, Long>{}
      ```
    - MemberService
      ```java
      public interface MemberService {	
          void validate(Long memberid) throws InvalidMemberException;
          Optional<Member> findById(Long memberId) throws MemberNotFoundException;
      }
      ```
    - StudyServiceTest
      ```java
      @ExtendWith(MockitoExtension.class)
      public class StudyServiceTest {
          @Mock MemberService memberService;
          @Mock StudyRepository studyRepository;

          @Test
          void createStudyService() {
              StudyService studyService = new StudyService(memberService, studyRepository);
              assertNotNull(studyService);
          }
      }
      ```
    - 다만, 이렇게 테스트 하면 추상메서드가 구현되지 않아 기능을 수행하지 않는다.
    - 기능 수행을 위해 stubbing 작업이 필요하다.
  
#### Mock 객체 Stubbing

- 모든 Mock 객체의 행동
  - Null을 리턴한다. (Optional 타입은 Optional.empty 리턴)
  - Primitive 타입은 기본 Primitive 값
  - 콜렉션은 비어있는 콜렉션
  - Void 메소드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.
  
- Mock 객체를 조작해서
  - 특정한 매개변수를 받은 경우 특정한 값을 리턴하거나 예외를 던지도록 만들 수 있다.
    - How about some stubbing?
      - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#2
    - Argument matchers
      - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#3  
  - Void 메소드 특정 매개변수를 받거나 호출된 경우 예외를 발생 시킬 수 있다.
    - Stubbing void methods with exceptions
      - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#5  
  - 메소드가 동일한 매개변수로 여러번 호출될 때 각기 다르게 행동하도록 조작할 수도 있다.
    - Stubbing consecutive calls
      - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#10
  
- 예제
  ```java
  @ExtendWith(MockitoExtension.class)
  public class StudyServiceTest {
      @Mock MemberService memberService;
      @Mock StudyRepository studyRepository;
      @Test
      void createNewStudy() {
          StudyService studyService = new StudyService(memberService, studyRepository);
          assertNotNull(studyService);

          Member member = new Member();
          member.setId(1L); member.setName("test");

          // stubbing - 해당 메서드가 호출 될 때 member를 optoinal하여 리턴
          when(memberService.findById(1L)).thenReturn(Optional.of(member));
          assertEquals("test", memberService.findById(1L).get().getName());
          // memberService.findByid(2L)이 호출되는 것은 허용되지 않는다. 
          // 파라미터를 any()로 넣으면 어떤 파라미터여도 같은 결과값을 얻을 수 있다.
          assertTrue(memberService.findById(2L).isEmpty());

          // 예외에 대한 stubbing
          doThrow(new IllegalArgumentException()).when(memberService).validate(1L);
          assertThrows(IllegalArgumentException.class, ()->{
              memberService.validate(1L);
          });

          // 반복적으로 매개변수를 여러번 호출할 때 각기 동작할 수 있다.
          when(memberService.findById(ArgumentMatchers.anyLong()))
              .thenReturn(Optional.of(member))
              .thenThrow(new IllegalArgumentException())
              .thenReturn(Optional.empty());
          assertEquals("test", memberService.findById(1L).get().getName());
          assertThrows(IllegalArgumentException.class, ()->{
              memberService.findById(2L);
          });
          assertTrue(memberService.findById(3L).isEmpty());
      }
  }
  ```

#### 객체 확인

- Mock 객체가 어떻게 사용이 됐는지 확인할 수 있다.
  - 특정 메소드가 특정 매개변수로 몇번 호출되었는지, 최소 한번은 호출됐는지, 전혀 호출되지 않았는지
    - Verifying exact number of invocations
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#exact_verification
  - 어떤 순서대로 호출했는지
    - Verification in order
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#in_order_verification
  - 특정 시간 이내에 호출됐는지
    - Verification with timeout
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#verification_timeout
  - 특정 시점 이후에 아무 일도 벌어지지 않았는지
    - Finding redundant invocations
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#finding_redundant_invocations

- 예제
  - MemberService.java
    ```java
    public void validate(Long memberid) throws IllegalArgumentException;
    public Optional<Member> findById(Long memberId) throws MemberNotFoundException;
    public void notice(Study study);
    public void notice(Member member);
    ```
  - StudyService.java
    ```java
    public Study createNewStudy(Long memberId, Study study) {
        Optional<Member> member = memberService.findById(memberId);
        study.setOwner(member.orElseThrow(()-> new IllegalArgumentException("Member doesn't exist for id: " + memberId)));
        Study newStudy = repository.save(study);
        memberService.notice(newStudy);
        return newStudy;
    }
    ```
    
  - 테스트 코드
    ```java
    @Test
    void testMockito() {
        Study study = new Study(10, "Test");
        Member member = new Member(); 
        member.setName("Test");

        when(memberService.findById(1L))
          .thenReturn(Optional.of(member));

        when(studyRepository.save(study))
          .thenReturn(study);

        StudyService studyService = new StudyService(memberService, studyRepository);
        studyService.createNewStudy(1L, study);
        assertNotNull(study.getOwner());
        assertEquals(member, study.getOwner());
        
        // verify를 통해서 메소드가 몇번 일어났는지 확인한다.
        verify(memberService, times(1)).notice(study);
        // verify(memberService, times(1)).notice(ArgumentMatchers.any()); 는 깨진다. 오버로딩된 notice가 존재하기 때문
        verify(memberService, never()).validate(ArgumentMatchers.any());
        
        // inOrder를 통해서 순서까지 확인할 수 있다.
        InOrder inOrder = Mockito.inOrder(memberService);
        inOrder.verify(memberService).findById(ArgumentMatchers.anyLong());
        inOrder.verify(memberService).notice(study);

        // memberService에서 더이상 호출되지 않는다는 의미
        verifyNoMoreInteractions(memberService);
    }  
    ```

#### BDD 스타일 API 

- BDD: 애플리케이션이 어떻게 "행동"해야 하는지에 대한 공통된 이해를 구성하는 방법으로, TDD에서 창안
  - https://en.wikipedia.org/wiki/Behavior-driven_development
  
- 행동에 대한 스펙
  - Title
  - Narrative
    - As a /I want / so that
  - Acceptance criteria
    - Given, When, Then
    
- Mockito는 BddMockito라는 클래스를 통해 BDD 스타일의 API를 제공한다.

- When-Then으로 구성된 예
  ```java
  // When
  studyService.createNewStudy(1L, study);
  
  // Then
  assertNotNull(study.getOwner());
  ```
  - createNewStudy를 실행하면 study의 owner가 null이 아니다.

- Mockito에서 제공하는 API가 when, thenReturn등으로 BDD의 When, Then 상황과 맞지 않아 새로운 API를 제공하고 있다.
  - When -> Given
    ```java
    given(memberService.findById(1L)).willReturn(Optional.of(member)); 
    // when(memberService.findById(1L)).thenReturn(Optional.of(member));
    
    given(studyRepository.save(study)).willReturn(study);
    // when(studyRepository.save(study)).thenReturn(study);
    ```
  - Verify -> Then
    ```java    
    then(memberService).should(times(1)).notify(study);
    // verify(memberService, times(1)).notify(study);
    then(memberService).shouldHaveNoMoreInteractions();
    // verifyNoMoreInteractions(memberService);
    ```

- 참고
  - https://javadoc.io/static/org.mockito/mockito-core/3.2.0/org/mockito/BDDMockito.html
  - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#BDD_behavior_verification
