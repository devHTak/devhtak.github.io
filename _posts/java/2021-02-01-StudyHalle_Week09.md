---
layout: post
title: 스터디 할래 9. 예외처리
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-02-01 21:41:00 +0900'
category: Java Study
---

#### 학습할 것

- 자바에서 예외 처리 방법 (try, catch, throw, throws, finally)
- 자바가 제공하는 예외 계층 구조
- Exception과 Error의 차이는?
- RuntimeException과 RuntimeException가 아닌 것의 차이는?
- 커스텀한 예외 만드는 방법

#### 자바에서 예외 처리 방법 (try, catch, throw, throws, finally)

- try catch finally 문

  ```java
  try {
      // 예외가 발생할 가능성이 있는 코드
  } catch (Exception1 e1) {
      // Exception1이 발생했을 때, 이를 처리하기 위한 코드
  } catch (Exception2 e2) {
      // Exception2가 발생했을 때, 이를 처리하기 위한 코드
  } catch (ExceptionN eN) {
      // ExceptionN이 발생했을 때, 이를 처리하기 위한 코드
  } finally {
      // 예외 발생여부와 상관없이 실행
  }
  ```
  - 여러개의 catch 블록이 있으며, 이 중 하나의 catch문이 실행된다.
  - 여러개의 catch 블록은 위에서 아래로 해당 exception에 포함되는 지 확인하여 실행된다. 즉, 첫번째 catch문에 최상위 Exception을 사용하면 하위 catch문을 거치는 예외는 없다.
  
- multi-check catch

  ```java
  try {
      // 예외 발생 가능성이 있는 코드
  } catch(Exception1 | Exception2 e2) {
      // Exception1 또는 Exception2 발생했을 때, 처리하는 코드
  }
  - JDK 1.7부터 지원한다.
  - OR 역할을 하여 작성된 모든 예외를 잡는다.
  - e.getClass() 로 발생한 예외 클래스를 알 수 있다.
  - Exception1과 Exception2가 상속관계인 경우 컴파일 오류가 발생한다. 
    - 자식클래스가 잡을 수 있는 예외는 부모클래스 또한 잡을 수 
    - 코드 중복이다.
    
- throw

  ```java
  public int divide(int num1, int num2) {
      if(num2 == 0)
          throw new IllegalArgumentException("0으로 나눌 수 없습니다.");
      
      return num1 / num2;
  }
  ```
  - 고의로 예외를 발생시킬 수 있다.
  - throw문이 있는 메소드를 사용하면, try-catch문으로 처리하거나 throws로 다음 호출 메서드에게 넘길 수 있다.

- throws

  ```java
  void mehtod() throws Exception1, Exception2, ..., Exception3 {
      // 본문
  }
  ```
  - throws를 통해 method에서 발생할 예외를 호출한 메서드에게 넘길 수 있다.
  - throws 키워드는 메서드에 에외를 선언할 수 있으며, 여러 개의 예외를 쉼표로 구분하여 선언할 수 있다. 
    
- finally
  - 예외 발생여부와 상관없이 실행한다.
  - try-catch-finally 순으로 실행한다.
  - 만약 try, catch 문 안에 return문이 있어 종료되어도 finally문은 실행한다.

- try-with-resource

  ```java
  try(FileOutputStream out = new FileOutputStream("thewing.txt")) { 
      // 본문
  } catch(IOException e){ 
      e.printStackTrace(); 
  }
  ```
  - Exception이 발생하는 경우 resource를 자동으로 close를 해준다.
    - finally에서 close해줄 필요가 없다.
  - 사용 로직을 작성할 때 객체는 AutoCloseable 인터페이스를 구연한 객체여야 한다.
  - try () 괄호 안에 ';'으로 여러 라인을 입력할 수 있다.
  - JDK 1.7부터 추가되었다.

#### 자바가 제공하는 예외 계층 구조

- Throwable
  - Exception과 Error 클래스는 Throwable 클래스를 상속받아 처리하도록 되어있다.
  - Exception이나 Error를 처리할 때 Throwable로 처리해도 무관하다
  - Throwable 클래스에 선언되어있고, Exception클래스에서 Overring한 메소드는 10개가 넘으며 가장 많이 사용되는 메소드는 getMassage, toString, printStackTrace가 있다.
  - 메소드
    - getMessage()
      - 예외 메시지를 String형태로 제공받는다.
      - 예외가 출력되었을 때 어떤 예외가 발생되었는지를 확인할 때 매우 유용하다.
      - 메시지를 활용하여 별도의 예외 메시지를 사용자에게 보여주려고 할 때 좋다.
    - toString()
      - 예외메시지를 String형태로 제공받는다.
    - getMessage()
      - 메소드보다는 약간 더 자세하게, 예외 클래스 이름도 같이 제공한다.
    - printStackTrace()
      - 가장 첫 줄에는 예외 메시지를 출력하고, 두 번째 줄부터는 예외가 발생하게 된 메소드들의 호출 관계(스택 트레이스를)출력해준다.
      - printStackTrace()는 정보 노출 가능성이 있기 때문에 서비스 운용시 사용하면 안된다

- Error
  - 컴퓨터 하드웨어의 오동작 또는 고장으로 인해 응용프로그램에 이상이 생겼거나 JVM 실행에 문제가 생겼을 경우 발생하는 것이다.
  - 프로세스에 영향을 준다
  - 시스템 레벨에서 발생한다(자바 프로그램 외의 오류)
  - 종류
    - VirtualMachineError
    - OutOfMemoryError
    - StackOverflowError
    - 등등

- Exception
  - 컴퓨터의 에러가 아닌 사용자의 잘못된 조작 또는 개발자의 잘못된 코딩으로 인해 발생하는 프로그램 오류이다.
  - 예외가 발생하면 프로그램이 종료가 된다는 것은 에러와 동일하지만 예외는 예외처리(Exception Handling)를 통해 프로그램을 종료되지 않고 정상적으로 작동되게 만들어줄 수 있다. 자바에서 예외처리는 Try Catch문을 통해 해줄 수 있다.
  - 개발자가 구현한 로직에서 발생
  - 쓰레드에 영향을 준다
  
- Checked Exception
  - 반드시 예외 처리 해야한다
  - 컴파일 단계에서 확인해야 한다.
  - 종류
    - RuntimeException를 제외한 모든 예외
    - IOException
    
- RuntimeException
  - 예외가 발생할 것을 미리 감지하지 못했을 때 발생.
  - 런타임 예외에 해당하는 모든 예외들은 RuntimeException을 확장한 예외들이다.

#### Exception과 Error의 차이는?

- Error vs Exception
  - Error는 OOM, StackOverflow 등 JVM, 하드웨어에서 발생하는 것이다. 발생하는 순간 무조건 프로그램은 비정상 종료되기 때문에 발생하지 않도록 유의해야 한다.
  - Exception은 프로그래머가 미리 적절한 코드를 작성하여 프로그램이 비정상 종료되지 않도록 핸들링할 수 있다.

#### RuntimeException과 RuntimeException가 아닌 것의 차이는?

- RuntimeException vs Non RuntimeException
  - 런타임(JVM 구동)시 예외가 발생하는 것이 UnCheckedException(RuntimeException)이고 실행하지 않고 예외가 검출되는 것이 CheckedException 이다

- RuntimeException
  - RuntimeException 클래스를 상속받는 자식 클래스들은 주로 치명적인 예외 상황을 발생시키지 않는 예외들로 구성되어있다.
  - try / catch문을 사용하기 보다는 프로그램을 작성하면서 예외가 발생하지 않도록 주의 하는 것이 좋다

- CheckedException
  - CheckedException 클래스인 Exception 클래스에 속하는 자식 클래스들은 치명적인 예외 상황을 발생시키기 때문에 반드시 try / catch 문을 사용하여 예외처리를 해야만한다
  - 자바 컴파일러는 RuntimeException 클래스 이외의 Exception 클래스의 자식 클래스에 속하는 예외가 발생할 가능성이 있는 구문에는 반드시 예외를 처리하도록 강제하고 있다
  - 만약 이러한 예외가 발생할 가능성이 있는 구문을 예외처리하지 않았을 때는 컴파일 시 오류를 발생시킨다.
  - 컴파일 단계에서 확인가능

#### 커스텀한 예외 만드는 방법

- 주의 
  - Throwable을 직접 상속 받는 클래스는 Exception과 Error가 있다
  - Error와 관련된 클래스는 개발자가 손대서는 절대 안된다*
  - Exception 을 처리하는 클래스라면 java.lang.Exception 클래스의 상속을 받는 것이 좋다.
  
```java
public class MyException extends Exception{
    public MyException(){
        super();
    }

    public MyException(String message){
        super(message);
    }
}
```
- 자바 예외 처리 전략
  - 예외를 어떻게 처리할지를 알아두는 것이 좋다
  - 실행시에 발생할 확률이 매우 높은 경우에는 런타임 예외로 만드는것이 나을 수도 있다. 
    - 즉, 클래스 선언시 extends Exception 대신에 extends RuntimeException 으로 선언하는 것이다. 
    - 이렇게 되면 해당 예외를 던지는(throw하는) 메소드를 사용하더라도 try-catch 로 묶지 않아도 컴파일시에 예외가 발생하지 않는다. 
    - 하지만, 이 경우에는 예외가 발생할 경우 해당 클래스를 호출하는 다른 클래스에서 예외를 처리하도록 구조적인 안전장치가 되어있어야만 한다. 
    - 여기서 안전 장치라고 하는 것은 try-catch 로 묶지 않은 메소드를 호출하는 메소드에서 예외를 처리하는 try-catch 가 되어 있는 것을 이야기한다
    
    ```java
    public void methodCaller() {
        try {
            methodCaller();
        } catch (Exception e) {
            // 예외처리
        }
    }
    public void methodCallee() {
        // RuntimeException 예외 발생 가능성 있는 부분분
    }
    ```
  - 이와 같이 unchecked exception인 RuntimeException이 발생하는 메소드가 있다면, 그 메소드를 호출하는 메소드는 try-catch 로 묶어주지 않더라도 컴파일할 때 문제가 생기지 않는다.
  - 하지만, 예외가 발생할 확률은 높으므로, 위의 예에서 methodCaller() 처럼 try-catch 로 묶어주는 것이 좋다
    ```java
    try {
        //예외발생 가능한코드
    } catch (SomeException e) {
        // 여기 아무 코드없음
    }
    ```
  - 이렇게 catch문장을 처리해주는건 피해야한다.
  - 여기서 SomeException 이라는 것은 그냥 어떤 예외를 잡는다는 것을 의미할뿐 실제 존재한다는게 아니다
  - catch에 아무런 작업을 하지않으면 어디서 발생했는지 전혀 문제를 찾을 수 없다.
