---
layout: post
title: \[더 자바, 코드를 조작하는 다양한 방법\] 리프렉션
summary: The Java
author: devhtak
date: '2021-01-23 14:41:00 +0900'
category: Java
---

#### 클래스 정보 조회

- 리플렉션의 시작은 Class<T>
  - https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html
  
- Class<T>에 접근하는 방법
  - 모든 클래스를 로딩한 다음 Class<T> 인스턴스가 생긴다. "타입.class"로 접근할 수 있다.
  - 모든 인스턴스는 getClass() 메소드를 가지고 있다. "인스턴스.getClass()"로 접근할 수 있다.
  - 클래스를 문자열로 읽어오는 방법
    - Class.forName("FQCN") // 풀패키지클래스네임
    - 클래스패스에 해당하는 클래스가 없다면 ClassNotFoundException이 발생한다.
    
    ```java
    public static void main(String[] args) throws ClassNotFoundException {
        Class<Book> bookClass = Book.class;

        Book book = new Book();
        Class<? extends Book> aClass = book.getClass();

        Class<?> bookClass2 = Class.forName("com.study.Book");
    } 
    ```
    
  - Class<T>를 통해 가져올 수 있는 것
    - 필드(목록) 가져오기
      ```java
      // public 만 가져온다
      Arrays.stream(bookClass.getFields()).forEach(System.out::println); 
      // 모든 필드 중 a 이름의 필드및 값을 가져온다.
      // accessible을 true로 선택하면 접근지시자를 무시할 수 있다.
      Book book = new Book();
      Arrays.stream(bookClass.getDeclaredFields()).forEach(f -> {
          try {
              f.setAccessible(true);
              System.out.println(String.format("%s %s", f, f.get(book)));

              int modifier = f.getModifiers();
              System.out.println(Modifier.isPrivate(modifier));
              System.out.println(Modifier.isStatic(modifier));
          } catch(IllegalAccessException e) {
              e.printStackTrace();
          }
      });
      ```
    - 메소드(목록) 가져오기
      ```java
      // 메소드
      Arrays.stream(MyBook.class.getMethods()).forEach(m -> {
          int modifier = m.getModifiers();
          System.out.println(Modifier.isAbstract(modifier));
      });
      ```
    - 상위 클래스 가져오기
      ```java
      // 상위 클래스를 가져올 수 있다.
		  System.out.println(MyBook.class.getSuperclass());
      ```
    - 인터페이스(목록) 가져오기
      ```java
      // 인터페이스
		  Arrays.stream(MyBook.class.getInterfaces()).forEach(System.out::println);
      ```
    - 애노테이션 가져오기
      ```java
      Arrays.stream(bookClass.getAnnotations()).forEach(System.out::println);
      ```
    - 생성자 가져오기
      ```java
      // 생성자 가져오기
		  Arrays.stream(bookClass.getConstructors()).forEach(System.out::println);
      ```
    - 등등등
    
#### Annotation과 Reflection

- 중요 Annotation
  ```java
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.TYPE, ElementType.FIELD})
  @Inherited
  public @interface MyAnnotation {
      String value(); // value는 속성설정할 때 이름을 할당한할 수 있다.
      int max() default 100;
  }
  ```
  - @Retention: 해당 애노테이션을 언제까지 유지할 것인가? 소스, 클래스, 런타임
    - RetentionPolicy.CLASS, RetentionPolicy.SOURCE, RetentionPolicy.RUNTIME
  
  - @Inherit: 해당 애노테이션을 하위 클래스까지 전달할 것인가
    - Book 클래스에 붙은 MyAnnotation이 하위 클래스인 MyBook까지 전달받는 것을 확인할 수 있다.
      ```java
      @MyAnnotation
      public class Book {}
      ```
      ```java
      public class MyBook extends Book {}
      ```
      ```java
      public static void main(String[] args) {
          Arryas.stream(MyBook.class.getAnnotations()).forEach(System.out:println); // @com.study.MyAnnotation(max=20, value="HELLO")
      }
      ```
    
  - @Target: 어디에 사용할 수 있는가?
    - ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.FIELD 등 Annotation을 사용할 수 있는 위치를 정하여 제한할 수 있다.
      
- Reflection
  - getAnnotation(): 상속받은 (@Inherit) 애노테이션까지 조회
    - 예제) fields에 붙은 Annotation이 MyAnnotation 타입인지 확인한 후 value와 number를 출력
      ```java
      public static void main(String args) {
          Arrays.stream(Book.class.getDeclaredFields()).forEach(f -> {
              Arrays.stream(f.getAnnotations()).forEach(a -> {
                  if(a instanceof MyAnnotation) {
                      MyAnnotation myAnnotation = (MyAnnotation)a;
                      System.out.println(myAnnotation.value());
                      System.out.println(myAnnotation.number());
                  }
              });
          });
      }
      ```
  - getDeclaredAnnotations(): 자기 자신에만 붙어있는 애노테이션 조회
  
