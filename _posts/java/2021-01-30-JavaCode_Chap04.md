---
layout: post
title: 더 자바, 코드를 조작하는 다양한 방법_다이나믹 프록시
summary: The Java
author: devhtak
date: '2021-01-30 14:41:00 +0900'
category: The Java
---

#### 스프링 데이터 JPA는 어떻게 동작하나?

- java.lang.reflect.Proxy 클래스
  - 인스턴스와 같이 행동하는 Proxy 객체를 리턴해준다.
    ```java
    InvocationHandler handler = new MyInvocationHanler(...);
    Foo f = (Foo)Proxy.newProxyInstance(Foo.class.getClassLoader(), new Class<?>[] {Foo.class}, handler);
    ```
- org.springframework.aop.framework.ProxyFactory 클래스
  - Java의 다이나믹 프록시를 좀 더 추상화하여 제공하는 스프링 AOP의 핵심 클래스  

- 스프링 데이터 JPA에서 인터페이스 타입의 인스턴스는 누가 만들어 주는 것인가?
  - Spring AOP 기반으로 동작하며 RepositoryFactorySupport에서 프록시를 생성한다.

#### 프록시 패턴

- 프록시와 리얼 서브젝트가 공유하는 인터페이스가 있고, 클라이언트는 해당 인터페이스 타입으로 프록시를 사용
- 클라이언트는 프록시를 거쳐 리얼 서브젝트를 사용
- 클라이언트는 프록시를 거쳐 리얼 서브젝트를 사용하기 때문에 프록시는 리얼 서브젝트에 대한 접근을 관리하거나 부가 기능을 제공하거나 리턴값을 변경할 수도 있다.
- 리얼 서브젝트는 자신이 해야 할 일만 하면서(SRP - Single Responsibility Principle) 프록시를 사용해서 부가적인 기능(접근 제한, 로깅, 트랜잭션 등)을 제공할 때 이런 패턴을 주로 사용한다.

- 예제
  - BookService: Interface로 서브젝트
    ```java
    public interface BookService {
	      void rent(Book book);
    }
    ```
  - DefaultBookService: BookService 구현체로 리얼 서브젝트
    ```java
    @Service
    public class DefaultBookService implements BookService{
        @Autowired
        public BookRepository bookRepository;

        @Override
        public void rent(Book book) {
            // TODO Auto-generated method stub
            System.out.println("Rent: " + book.getTitle());
        }
    }
    ```
  - BookServiceProxy: Proxy로 클라이언트는 서브젝트
    - BookService를 사용할 때 프록시인 BookServiceProxy를 사용하도록 되어 있다.
    - BookServiceProxy가 리얼 서브젝트인 DefaultBookService를 사용한다.
    ```java
    @Service
    public class BookServiceProxy implements BookService {
        // Qualifier 빈의 가져올 이름(클래스명)
        @Autowired @Qualifier("defaultBookService") 
        private BookService bookService;

        @Override
        public void rent(Book book) {
            // TODO Auto-generated method stub
            System.out.println("AAA");
            bookService.rent(book);
            System.out.println("BBB");
        }
    }
    ```
  - Test
    ```java
    @SpringBootTest
    public class BookServiceProxyTest {
        @Autowired @Qualifier("bookServiceProxy")
        private BookService bookService;

        @Test
        public void test() {
            Book book = new Book();
            book.setTitle("spring");
            bookService.rent(book);
        }
    }
    ```  

- 참고
  - https://www.oodesign.com/proxy-pattern.html
  - https://en.wikipedia.org/wiki/Proxy_pattern
  - https://en.wikipedia.org/wiki/Single-responsibility_principle  

#### 다이나믹 프록시 실습

- 런타임에 특정 인터페이스들을 구현하는 클래스 또는 인스턴스를 만드는 기술
- an application can use a dynamic proxy class to create an object that implements multiple arbitary event listener interfaces
- 프록시 인스턴스 만들기
  - Object Proxy.newProxyInstance(ClassLoader, Interfaces, InvocationHandler)
    - 프록시 패턴을 만들지 않고도 구현이 가능하다.
    - 인터페이스(서브젝트)와 구현 클래스(리얼 서브젝트)만 있으면 된다.
    - InvocationHandler에서 invoke를 오버라이딩하여 프록시 설정
  - 하지만 유연한 구조가 아니다. 그래서 스프링 AOP 등장
  - 스프링 AOP에 대한 더 자세한 사항은 토비의 스프링 AOP 참고
    ```java
    @SpringBootTest
    public class BookServiceProxyTest {
        BookService bookService = (BookService)Proxy.newProxyInstance(BookService.class.getClassLoader(), 
            new Class[] {BookService.class}, 
            new InvocationHandler() {
                BookService bookService = new DefaultBookService();
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    // TODO Auto-generated method stub
                    // if(method.getName().equals("rent") {...} 로 메소드별로 수정할 수 있다.
                    System.out.println("aaaa");
                    Object invoke = method.invoke(bookService, args);
                    System.out.println("bbbbb");

                    return invoke;
                }
            });
        @Test
        public void test() {
            Book book = new Book();
            book.setTitle("spring");
            bookService.rent(book);
        }
    }   
    ```
  
- 참고
  - https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html
  - https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html#newProxyInstance-java.lang.ClassLoader-java.lang.Class:A-java.lang.reflect.InvocationHandler-
  
#### 클래스의 프록시가 필요하다면?

- 서브 클래스를 만들 수 있는 라이브러리를 사용하여 프록시를 만들 수 있다.
- CGlib
  - https://github.com/cglib/cglib/wiki
  - 스프링, 하이버네이트가 사용하는 라이브러리
  - 버전 호환성이 좋지 않아서 서로 다른 라이브러리 내부에 내장된 형태로 제공되기도 한다.
  - dependency
    ```
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>3.3.0</version>
    </dependency>
    ```
  - 예제
    - 인터페이스(서브젝트) 없이 구현 가능 
    ```java
    @SpringBootTest
    public class BookServiceTest {
        @Test
        public void di() {
            MethodInterceptor handler = new MethodInterceptor() {
                BookService bookService = new BookService();
                @Override
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    if(method.getName().equals("rent") {
                        System.out.println("aaaa");
                        Object invoke = method.invoke(bookService, objects);
                        System.out.println("bbbb");
                        return invoke;
                    }
                    return method.invoke(bookService, objects);
                }
            }
            BookService bookService = (BookService)Enhancer.create(BookService.class, handler);
            bookService.rent();
        }
    }
    ```
- ByteBuddy
  - https://bytebuddy.net/#/
  - 바이트 코드 조작뿐 아니라 런타임(다이나믹) 프록시를 만들 때도 사용할 수 있다.
  - 예제
    ```java
    @SpringBootTest
    public class BookServiceTest {
        @Test
        public void di() throws Exception{
            Class<? extends BookService> proxyClass = new ByteBuddy().subclass(BookService.class)
                .method(named("rent")).intercept(InvocationHandlerAdapter.of(new InvocationHandler() {
                    BookService bookService = new BookService();
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        // TODO Auto-generated method stub
                        // method에서 rent이름만 가져와 intercept하기 때문에 분기탈 필요가 없다
                        System.out.println("aaaa");
                        Object invoke = method.invoke(bookService, args);
                        System.out.println("bbbbb");
                        return invoke;
                    }
                }).make().load(BookService.class.getClassLoader()).getLoaded();
            BookService bookService = proxyClass.getCosntructor(null).newInstance();
        }
    }
    ```
  
- 서브 클래스를 만드는 방법의 단점
  - 상속을 사용하지 못하는 겨우 프록시를 만들 수 없다.
    - private 생성자만 있는 경우
    - Final 클래스인 경우
  - 인터페이스가 있을 때에는 인터페이스의 프록시를 만들어 사용할 것

#### 다이나믹 프록시 정리

- 다이나믹 프록시
  - 런타임에서 인터페이스 똔느 클래스의 프록시 인스턴스 또는 클래스를 만들어 사용하는 프로그래밍 기법

- 다이나믹 프록시 사용처
  - 스프링 데이터 JPA
  - 스프링 AOP
  - Mockito
  - 하이버네이트 lazy initialization
  - ...
  
- 참고
  - http://tutorials.jenkov.com/java-reflection/dynamic-proxies.html
  
