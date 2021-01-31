---
layout: post
title: 더 자바, 코드를 조작하는 다양한 방법_리프렉션
summary: The Java
author: devhtak
date: '2021-01-23 14:41:00 +0900'
category: The Java
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
	// Class aClass = (Class<Book>)bok.getClass();
	
        Class<?> bookClass2 = Class.forName("com.study.Book");
	// Class<Book> bookClass2 = (Class<Book>)Class.forName("com.study.Book");
    } 
    ```
    
  - Class<T>를 통해 가져올 수 있는 것
    - 필드(목록) 가져오기
      - class.getFields() : public 메서드만 가져온다.
      - class.getDeclaredFields(): 접근지시자에 상관없이 가져온다. (field에 setAccessible을 true로 주면 접근지시자에 접근을 가능하도록 한다.) 

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
      - getMethods(): 부모 객체에서 선언한 method도 가져온다.
      - getDeclaredMethods(): 해당 객체에 선언된 method만 가져온다.
      
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
  
  - @Inherited: 해당 애노테이션을 하위 클래스까지 전달할 것인가
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
  - getAnnotation(): 상속받은 (@Inherited) 애노테이션까지 조회
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
  
#### 클래스 정보 수정

- Class 인스턴스 만들기
  - Class.newInstance()는 deprecated 됐으며, 이제부터는 생성자를 통해서 만든다.
  
- 생성자로 인스턴스 만들기
  - Constructor.newInstance(params)
  
- 필드 값 접근하기/설정하기
  - 특정 인스턴스가 가지고 있는 값을 가져오는 것이기 때문에 인스턴스가 필요하다.
  - Field.get(object)
  - Field.set(object, value)
  - static 필드를 가져올 때는 object가 없어도 되니까 null을 넘기면 된다.
  
- 메소드 실행하기
  - Object Method.invoke(object, params)

- 예제

  ```java
  Class<?> bookClass = Class.forName("com.study.Book");
  Constructor<?> constructor = bookClass.getConstructor(String.class);
  Book book = (Book)constructor.newInstance("myBook");
  System.out.println(book);
  
  Field f = Book.class.getDeclaredField("B");
  f.setAccessible(true);
  System.out.println(f.get(book)); // myBook
  
  f.set(book, "myBook2");
  System.out.println(f.get(book)); // myBook2
  
  Method method = Book.class.getDeclaredMethod("sum", int.class, int.class);
  method.setAccessible(true);
  int invoke = (int)method.invoke(book, 1, 2); // book의 C 메소드 실행, return 받을 수 있고, 캐스팅 가능
  System.out.println(invoke); // 3
  ```
  
#### 나만의 DI 프레임워크 만들어보기

- @Inject라는 애노테이션을 만들어서 필드 주입해주는 컨테이너 서비스 만들기

  ```java
  public class BookService {
      @Inject
      BookRepository bookRepository
  }
  ```
  
- ContainerService.java
  - classType에 해당하는 타입의 객체를 만들어 준다.
  - 단, 해당 객체의 필드 중에 @Inject가 있다면, 해당 필드도 같이 만들어 제공한다.
```java
private static <T> T createInstance(Class<T>  classType) {
	try {
		return classType.getConstructor().newInstance();
	} catch (InstantiationException | IllegalAccessException | IllegalArgumentException | InvocationTargetException
		| NoSuchMethodException | SecurityException e) {
		// TODO Auto-generated catch block
		throw new RuntimeException(e);
    	}
}
public static <T> T getObject(Class<T> classType) {
	T instance = createInstance(classType);
	Arrays.stream(classType.getDeclaredFields()).forEach(field -> {
		if(field.getAnnotation(Inject.class) != null) {
			Object fieldInstance = createInstance(field.getType());
			field.setAccessible(true);
			try {
				field.set(instance, fieldInstance);
			} catch (IllegalArgumentException | IllegalAccessException e) {
				// TODO Auto-generated catch block
				throw new RuntimeException();
			}
		}
    	});
    	return instance;
}
```
  - 테스트 코드
  
    ```java
    @Test
    public void getObjectTest() {
        BookRepository bookRepository = ContainerService.getObject(BookRepository.class);
        assertNotNull(bookRepository);
    }	
    @Test
    public void getObjectInjectionTest() {
        BookService bookService = ContainerService.getObject(BookService.class);
        BookRepository bookRepository = bookService.bookRepository;
        assertNotNull(bookRepository);
    }
    ```

#### 정리

- 리플렉션 사용시 주의할 점
  - 지나친 사용은 성능 이슈를 야기할 수 있다. 반드시 필요할 경우에만 사용
  - 컴파일 타임에 확인되지 않고 런타임 시에만 발생하는 문제를 만들 가능성이 있다.
  - 접근 지시자를 무시할 수 있다.
  
- 스프링
  - 의존성 주입
  - MVC 뷰에서 넘어온 데이터를 객체에 바인딩할 때
  
- 하이버네이트
  - @Entity 클래스에 Setter가 없다면 리플렉션을 사용한다.
  
- JUnit
  - https://junit.org/junit5/docs/5.0.3/api/org/junit/platform/commons/util/ReflectionUtils.html
  
- 참고
  - https://docs.oracle.com/javase/tutorial/reflect/index.html
  
