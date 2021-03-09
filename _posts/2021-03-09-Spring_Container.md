---
layout: post
title: Spring Container and Bean
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-09 21:41:00 +0900'
category: Spring
---

#### Spring Container

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

- ApplicationContext를 스프링 컨테이너라 한다.
- 기존에는 개발자가 AppConfig를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제는 스프링 컨테이너를 통해서 사용한다.
  ```java
  MemberService memberServiceBefore = appConfig.memberService(); // 기존 방식
  MemberService memberServiceAfter = context.getBean("memberService", MemberService.class); // Spring Container활용
  ```
- 스프링 컨테이너는 @Configuration이 붙은 AppConfig를 설정(구성) 정보로 사용한다.
  - 여기서 @Bean이라 적힌 메서드를 모두 호출하여 반환된 객체를 스프링 컨테이너에 등록
  - 이렇게 스프링 컨테이너에 등록된 객체를 Spring Bean이라 한다.
- 스프링 빈은 @Bean이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. (memberService, orderService)
- 이전에는 개발자가 필요한 객체를 AppConfig를 사용해서 직접 조회했지만, 이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다.
  - 스프링 빈은 applicationContext.getBean() 메서드를 사용해여 찾을 수 있다.
- 기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경하였다.


#### Spring Container 생성

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```
- ApplicationContext는 인터페이스이다.
- Spring Container는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.
- 직전에 AppConfig를 사용했던 방식이 애노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것
- 자바 설정 클래스를 기반으로 스프링 컨테이너(ApplicationContext)를 만들어보자
  - new AnnotationConfigApplicationContext(AppConfig.class);
  - 해당 클래스는 ApplicationContext 인터페이스의 구현체다.

- 스프링 컨테이너 생성 과정
  - 스프링 컨테이너 생성
    - Spring Container 생성(AppConfig.class) 
    - Spring Container는 AppConfig.class에 구성정보를 활용하여 스프링 빈 저장소를 만든다.
    - Spring Container 안에 있는 스프링 빈 저장소에는 빈 이름과 빈 객체로 key, value 형태로 저장된다.
  - 스프링 빈 등록
    - 스프링 컨테이너에는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.
    - 빈 이름
      - 빈 이름은 기본 메서드 이름을 사용한다.
      - 빈 이름을 직접 부여할 수도 있다. ex) @Bean(name="memberService")
      - 주의!! 빈 이름은 항상 다른 이름으로 부여해야 한다. 같은 이름을 부여하면 다른 빈이 무시되거나 기존 빈을 덮어버리거나 설정에 따라 오류 발생
  - 스프링 빈 의존관계 설정 - 준비
  - 스프링 빈 의존관계 설정 - 완료
    - 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입한다.
    - 단순히 자바 코드를 호출하는 것 같지만, 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 설명한다.
    - ex) memberService -> memberRepository <- orderService -> discountPolicy
      - memberService에서 memberRepository를 생성한다.
      - orderService는 memberRepository와 discountPolicy를 생성한다.
  
- 스프링 빈 의존관계 설정 준비 및 완료에서 참고할 부분
  - 스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나눠져 있다.
  - 그런데 이렇게 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다.
  - 여기서는 개념적으로 나누어 설명했다.
  
#### 컨테이너에 등록된 모든 빈 조회

```java
public class ApplicationContextInfoTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    @Test
    @DisplayName("모든 빈 조회")
    void findAllBean() {
        String[] beanName = ac.getBeanDefinitionNames();

        Arrays.stream(beanName).forEach((b) -> {
            Object bean = ac.getBean(b);
            System.out.println("name: " + b + ", object: " + bean);
        });
    }

    @Test
    @DisplayName("애플리케이션 빈 조회")
    void findApplicationBean() {
        String[] beanName = ac.getBeanDefinitionNames();

        Arrays.stream(beanName).forEach((b) -> {
            BeanDefinition beanDefinition = ac.getBeanDefinition(b);
            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(b);
                System.out.println("name: " + b + ", object: " + bean);
            }
        });
    }
}
```
- findAllBean() -> 모든 빈 출력하기
  - 실행하면 스프링에 등록된 모든 빈 정보를 출력할 수 있다.
  - ac.getBeanDefinitionNames(): 스프링에 등록된 모든 빈 이름을 조회한다.
  - ac.getBean(): 빈 이름으로 빈 객체(인스턴스)를 조회한다.

- findApplicationBean() -> 애플리케이션 빈 출력하기
  - 스프링이 내부에서 사용하는 빈은 제외하고, 내가 등록한 빈만 출력
  - 스프링이 내부에서 사용하는 빈은 getRole()로 구분할 수 있다.
    - BeanDefinition.ROLE_APPLICATOIN: 일반 사용자가 정의한 빈
    - BeanDefinition.ROLE_INFRASTRUCTURE: 스프링 내부에서 사용하는 빈

#### 스프링 빈 조회 - 기본

```java
public class ApplicationContextInfoTest {	
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);	
    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName() {
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회 실패")
    void findBeanByNameFailed() {
        assertThrows(NoSuchBeanDefinitionException.class, ()-> {
            ac.getBean("xxx", MemberService.class);
        });
    }

    @Test
    @DisplayName("이름없이 타입으로 조회")
    void findBeanByType() {
        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByDefiniteType() {
        MemberServiceImpl memberServiceImpl = ac.getBean(MemberServiceImpl.class);
        assertNotNull(memberServiceImpl);
    }
}
```

- 스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법
  - ac.getBean(빈이름, 타입)
  - ac.getBean(타입)
  - 조회 대상 스프링이 빈이 없으면 예외 발생
    - NoSuchBeanDefinitionException: No bean named 'XXXX' available

#### 스프링 빈 조회 - 동일한 타입이 둘 이상

```java
public class ApplicationContextInfoTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    
    @Test
    @DisplayName("타입 조회 시 중복 타입이 있는 경우 오류 발생")
    void findBeanByTypeDuplicate() {
        ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
        assertThrows(NoUniqueBeanDefinitionException.class, ()-> {
            ac.getBean(MemberRepository.class);
        });
    }

    @Test
    @DisplayName("타입 조회 시 중복 타입이 있는 경우 이름으로 조회")
    void findDuplicateBeanByName() {
        ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
        MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        assertNotNull(memberRepository);
    }

    @Test
    @DisplayName("특정 타입 모두 조회")
    void findAllDuplicateBeans() {
        ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

        Map<String, MemberRepository> memberRepositories = ac.getBeansOfType(MemberRepository.class);
        memberRepositories.keySet().forEach((name) -> {
            System.out.println("key: " + name + ", bean: " + memberRepositories.get(name));
        });

        assertEquals(2, memberRepositories.size());
    }

    @Configuration
    static class SameBeanConfig {
        @Bean 
        public MemberRepository memberRepository1() {
            return new InMemoryMemberRepository();
        }
        @Bean
        public MemberRepository memberRepository2() {
            return new InMemoryMemberRepository();
        }
    }
}
```

- 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정하자.
- ac.getBeanOfType()을 사용하면 해당 타입의 모든 빈을 조회할 수 있다. return 타입은 Map<String, Object>


- 출처: 김영한님의 스프링 핵심 기본편 강의 및 
