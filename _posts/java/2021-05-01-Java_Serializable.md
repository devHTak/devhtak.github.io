---
layout: post
title: Serializable
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-05-01 21:41:00 +0900'
category: Java Study
---

#### Java 직렬화란?

- 직렬화: 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술
  - Java Object -> Byte Array -> File or DB or Network
- 역직렬화: 바이트로 변환된 데이터를 다시 객체로 변환하는 기술
  - File or DB or Network -> Byte Array -> Java Object
- 즉, JVM에의 메모리(힙 또는 스택)에 있는 객체 데이터를 바이트 형태로 변환하는 기술과 데이터를 객체로 변환하여 JVM으로 상주시키는 형태

- serialVersionUID
  - serialVersionUID값을 저장할 때 클래스 버전이 맞는지 확인하기 위한 용도입니다.
    - 직렬화: Java Object -> Byte Array로 변환 당시 serialVersionUID 도 저장
    - 역직렬화: Byte Array -> Java Object로 변환 당시 serialVersionUID 체크, 다른 경우 Exception 발생
  - 만약 직렬화할 때 사용한 serialVersionUID의 값과 역직렬화 하기 위해 사용했던 serialVersionUID값이 다르다면 InvalidClassException이 발생할 수 있습니다.
  
- InvalidClassException 발생
  - 클래스의 Serial 버전이 다른 경우
  - 알 수 없는 데이터 타입을 포함한 경우
  - 기본 생성자가 없는 경우

#### 직렬화 사용방법

- Serializable을 구현하여 사용한다.
  ```java
  public class Member implements Serializable{	
      private String name;

      private int age;

      public Member() {}
      public Member(String name, int age) {
          this.name = name;
          this.age = age;
      }

      @Override
      public String toString() {
          // TODO Auto-generated method stub
          return String.format("Member name: %s, age: %s", this.name, this.age);
      }	
  }
  ```
  
- 직렬화 및 역직렬화
  ```java
  public static void main(String[] args) {
      Member memberTest01 = new Member("TEST01", 10);
      Member memberTest02 = new Member("TEST02", 20);

      byte[] serializedMember = null;

      // 직렬화
      try(ByteArrayOutputStream baos = new ByteArrayOutputStream() ) {
          try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
              oos.writeObject(memberTest01);
              oos.writeObject(memberTest02);

              serializedMember = baos.toByteArray();
          }
      } catch(IOException e) {
          e.printStackTrace();
      }

      // 역직렬화
      try(ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember) ) {
          try (ObjectInputStream ois = new ObjectInputStream(bais)) {
              Member returnMemberTest01 = (Member)ois.readObject();
              Member returnMemberTest02 = (Member)ois.readObject();
              System.out.println(returnMemberTest01);
              System.out.println(returnMemberTest02);
          }
      } catch(IOException e) {
          e.printStackTrace();
      }		
	}
  ```
  - 출력 값
    ```
    Member name: TEST01, age: 10
    Member name: TEST02, age: 20
    ```

- 파일에 쓰고(직렬화) 읽어오기(역직렬화)
  ```java
  public static void main(String[] args) {
      Member memberTest01 = new Member("TEST01", 10);
      Member memberTest02 = new Member("TEST02", 20);

      // 직렬화
      try(ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("testMember.txt"))) {
          oos.writeObject(memberTest01);
          oos.writeObject(memberTest02);
      } catch(IOException e) {
          e.printStackTrace();
      }

      // 역직렬화
      try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("testMember.txt"))) {
          Member returnMemberTest01 = (Member)ois.readObject();
          Member returnMemberTest02 = (Member)ois.readObject();
          System.out.println(returnMemberTest01);
          System.out.println(returnMemberTest02);
      } catch(IOException e) {
          e.printStackTrace();
      } catch(ClassNotFoundException e) {
          e.printStackTrace();
      }
	}
  ```
  - 파일에 쓰인 내용
    ```
    ы sr "com.naver.exam.serializable.MemberV?3봎~ I ageL namet Ljava/lang/String;xp   t TEST01sq ~     t TEST02
    ```
  - 읽어온 내용(console)
    ```
    Member name: TEST01, age: 10
    Member name: TEST02, age: 20
    ```
    
#### serialVersionUID 가 없을 때, 무슨 문제가 발생할까?

- 위 예제에서 serialVersionUID 를 작성하지 않아도 정상 동작했다.
- 하지만 요구사항이 추가되어 Member에 address 인스턴스 변수를 추가했다.
  ```java
  public class Member implements Serializable{
      private String name;

      private int age;

      private String address;

      public Member() {}
      public Member(String name, int age) {
          this.name = name;
          this.age = age;
      }
      
      public Member(String name, int age, String address) {
          this.name = name;
          this.age = age;
          this.address = address;
      }
      
      @Override
      public String toString() {
          // TODO Auto-generated method stub
          return String.format("Member name: %s, age: %s, address: %s", this.name, this.age, this.address);
      }	
  }
  ```
- 변경 전에 저장되어 있던 testMember.txt에서 읽어오면 address는 null로 제대로 읽어올까?
  ```java
  // 역직렬화
  try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("testMember.txt"))) {
      Member returnMemberTest01 = (Member)ois.readObject();
      Member returnMemberTest02 = (Member)ois.readObject();
      System.out.println(returnMemberTest01);
      System.out.println(returnMemberTest02);
  } catch(IOException e) {
      e.printStackTrace();
  } catch(ClassNotFoundException e) {
      e.printStackTrace();
  }
  ```
  - 하지만, java.io.InvalidClassException 가 발생하며 제대로 읽어오지 못했다.

- 이유
  - serialVersionUID를 명시하지 않는 경우 클래스의 기본 해쉬값을 사용한다.
  - 메소드, 변수 추가-삭제, 타입 변경등 클래스가 변경되는 경우 serialVersionUID가 변경되기 때문에 일치하지 않는 클래스로 인식하여 InvalidClassException이 발생

#### 결론
- serialVersionUID를 명시하고 관리하여 해당 이슈를 해결해야 한다.
- 변경이 잦은 클래스 정보에 경우 직렬화를 사용하지 않는 것이 좋다.
- 기본적으로 직렬화시에 기본적으로 타입에 대한 정보 등 클래스의 메타 정보도 갖고 있기 때문에 상대적으로 다른 포맷에 비해 용량이 크다.
  
** 참고[직렬화]: 우아한형제들, 자바 직렬화, 그것이 알고싶다. 훑어보기편
** 참고[직렬화]: 우아한형제들, 자바 직렬화, 그것이 알고싶다. 실무편
** 참고[serialVersionUID]: https://ktko.tistory.com/entry/JAVA-%EA%B0%9D%EC%B2%B4%EC%9D%98-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-serialVersionUID
** 참고[InvalidClassException]: https://tomining.tistory.com/80

