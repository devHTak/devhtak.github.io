---
layout: post
title: Effective Java Ch12. 직렬화
summary: Effective Java 3판 공부
author: devhtak
date: '2022-08-07 22:41:00 +0900'
category: Effective Java
---

#### Chap 12. 직렬화

- 직렬화란 자바 객체를 바이트 스트림으로 인코딩(직렬화) 하고, 해당 바이트 스트림을 다시 자바 객체로 재구성하는 (역직렬화) 메커니즘
- 직렬화한 바이트 스트림은 다른 VM에 전송하나 디스크에 저장한 후 나중에 역직렬화할 수 있다

##### Item85. 자바 직렬화의 대안을 찾아라

- 직렬화의 근본적인 문제는 공격 범위가 너무 넓고, 지속적으로 넓어진다는 점이다
  - ObjectInputStream의 readObject 메서드를 호출하면 객체 그래프가 역직렬화 된다
  - readObject 메서드는 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있다. 즉, 그 타입들의 코드 전체가 공격 범위에 들어간다는 것이다
    - 서드파티 라이브러리, 자신의 클래스들이 보안에 취약할 수 있다.

- 가젯
  - 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드들
  - 여러 가젯을 함께 사용하는 것을 가젯 체인이라 한다
  - 가끔 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있는 아주 강력한 가젯 체인이 발견되기도 한다

```java
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo");
        s1.add(t1);
        s1.add(t2);
        s2.add(t1);
        s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // 직렬화 수행
}
```
  - 객체 그래프를 살펴보면 201개의 HashSet으로 구성되어 있으며 그 각각은 3개 이하의 객체 참조를 갖는다.
  - 스트림 전체 크기는 5744byte 이지만, 역직렬화는 끝나지 않는다

- 해결 방법
  - 가장 좋은 방법은 아무것도 역직렬화하지 않는 것
    - JSON과 Protocol Buffers 처럼 다양한 메커니즘을 사용하는 것이 좋다
  - 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없다면 java 9에 나온 ObjectInputFilter를 사용하자 
    - 이는 데이터 스트림이 역직렬화되기 전에 필터를 적용해서 특정 클래스를 받아들이거나 거부할 수 있다
    - 블랙리스트/화이트리스트 방식이 있으며 화이트리슽트 방식을 추천

##### Item86. Serializable 을 구현할지는 신중히 결정하라

- 직렬화를 할 수 있게 하려면 클래스 선언에 implements Serializable 을 붙이면 된다
- 만들기는 쉽지만 만든 후가 문제다
  - Serializable 을 구현하면 릴리스한 뒤에 수정하기 어렵다
    - Serializable 을 구현하면 하나의 공개 API가 되기 때문에 해당 클래스가 널리 퍼진다면 그 직렬화 형태도 공개 API 처럼 지원해야 한다
      - 필드를 추가/삭제하는 경우 serialVersionUID 라는 이름의 static field long 필드로 명시해야된다
      - 그렇지 않은 경우 InvalidClassException 이 발생하여 쉽게 호환성이 깨지게 된다
    - private, package-private 인스턴스 필드마저 API로 공개되는 꼴이다(캡슐화)
  - Serializable 버그와 보안 구멍이 생길 위험이 높아진다
    - 역직렬화는 일반 생성자의 문제가 그대로 적용되는 숨은 생성자다
    - 불변식에 깨짐, 캡슐화 등이 깨질 수 있다
  - 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다
    - 신버전 인스턴스를 직렬화한 후, 구버전으로 역직렬화할 수 있는 지 등 테스트 사안이 늘어난다.
  - 구현 여부를 가볍게 판단해서는 안된다
  
- 상속용 클래스는 대부분 Serializable을 구현하면 안되며, 인터페이스도 대부분 Serializable 확장해서는 안된다
  - 하위 클래스, 인터페이스 구현 클래스에게 큰 부담을 주게 된다
  - 인스턴스 필드 값 중 불변식을 보장해야 한다면 하위 클래스에서 finalize 메서드를 재정의하지 못하게 해야한다
    - finalize 메서드를 재정의하여 final로 선언하면 막을 수 있다
    - 막지 않는다면 finalizer 공격에 취약해진다
  - 인스턴스 필드 중 기본값으로 초기화되면 불변식이 깨질경우 클래스에 readObjectNoData 메서드를 추가해야 한다

- 내부 클래스는 직렬화를 구현하지 말아야 한다
  - 바깥 인스턴스의 참조와 유효 범위 내 지역변수들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다
  - 내부 클래스에 대한 기본 직렬화 형태가 분명하지 않다
  - 정적 멤버 클래스의 경우 Serializable을 구현해도 된다

- Serializalbe 구현 여부 결정하는 방법
  - 객체를 전송하거나 저장할 때 직렬화를 이용하는 프레임워크를 사용하는 경우
  - 전통적으로 값 클래스와 컬렉션이 Serializable을 구현하고 스레드 풀처럼 동작하는 객체를 표현하는 클래스는 구현하지 않았다

- Serializalbe 구현하지 않을 때 주의점
  - 상속용 클래스가 직렬화를 지원하지 않고 하위 클래스에서 직렬화를 지원하려 할 때 부담이 크다
  - 보통 이런 클래스를 역직렬화시 상위 클래스가 매개변수 없는 생성자를 지원해야 한다
  - 생성자를 제공하지 않는다면 직렬화 프록시 패턴(아이템 90)을 사용해야 한다

##### Item87. 커스텀 직렬화 형태를 고려해보라

- 기본 직렬화 형태를 사용하기 위해서는 유연성, 성능, 정확성 측면에서 고민한 후 합당할 경우 사용해야 한다
- 객체의 물리적 표현과 논리적 표현이 같다면 기본 직렬화 형태로 제공해도 무방하다
  ```java
  public class Name implements Serializable {
      private final String firstName;
      private final String lastName;
      private final String middleName;
      // ...
  }
  ```
  - 기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다
    - readObject 에서 firstName, lastName 필드가 null 이 아님을 보장해야 한다

- 기본 직렬화가 적합하지 않은 케이스
  ```java
  public final class StringLsit implements Serializable {
      private int size = 0;
      private Entry head = null;
      
      public static class Entry implements Serializable {
          String data;
          Entry next;
          Entry previous;
      }
      // ... 나머지 생략
  }
  ```
  - 해당 클래스는 문자열을 이중 연결 리스트로 연결했다.
  - 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리를 철두철미하게 기록해야 한다
  - 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하는 경우 발생하는 문제
    - 공개 API 가 현재의 내부 표현 방식에 영구히 묶인다
    - 너무 많은 공간을 차지할 수 있따
    - 시간이 너무 많이 걸릴 수 있다
    - 스택 오버 플로우를 일으킬 수 있다
  - 합리적인 직렬화 형태
    ```java
    public final class StringList implements Serializable {
        private transient int size = 0;
        private transient Entry head = null;

        // 이제는 직렬화되지 않는다.
        private static class Entry {
            String data;
            Entry next;
            Entry previous;
        }

        // 지정한 문자열을 이 리스트에 추가한다.
        public final void add(String s) {...}

        /**
         * 이 {@code StringList} 인스턴스를 직렬화한다.
         * 
         * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
         * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
         * 순서대로 기록한다.
         */
        private void writeObject(ObjectOutputStream s) throws IOException {
          //기본 직렬화를 수행한다.
            s.defaultWriteObject();
            s.writeInt(size);

            // 커스텀 역직렬화를 수행한다.
            // 모든 원소를 올바른 순서로 기록한다.
            for (Entry e = head; e != null; e = e.next)
                s.writeObject(e.data);
        }

        private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
            //기본 역직렬화를 수행한다.
            s.defaultReadObject();
            int numElements = s.readInt();

            // 커스텀 역직렬화 부분
            // 모든 원소를 읽어 이 리스트에 삽입한다.
            for(int i = 0; i < numElements; i++) {
                add((String) s.readObject());
            }
        }
    }
    ```
    - writeObject, readObject 메서드가 직렬화 형태를 처리한다
    - transient 를 통해 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는 것을 나타낸다
      - defaultReadObject(), defaultWriteObject()
      - 모든 인스턴스 필드가 transient 이면, 호출하지 않아도 되지만 언제 필드가 추가될 지 모르기 때문에 호출
      - 구버전의 경우 역직렬화 할 때 StreamCorruptedException 이 발생한다
    - 역직렬화하면 불변식까지 포함해 제대로 복원해낸다는 점에서 정확하다고 할 수 있지만, 불변식이 세부 구현에 따라 달라지는 객체에서는 해당 정확성이 깨질 수 있다
      - 예제) 해쉬테이블 
      - 물리적으로 키-값 엔트리들을 담은 해시 버킷을 차례로 나열한 형태
      - hashCode 계산 방식에 따라 어떤 버킷에 담을지 나눠지게 되는데, 구현에 따라 달라지게 된다
      - 즉, HashTable을 기본 직렬화를 사용하면 역직렬화할 때 불변식이 훼손된 객체들이 발생할 수 있다.

- transient 와 defaultWriteObject
  - defaultWriteObject 를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화된다
  - 따라서 transient로 선언해도 되는 인스턴스 필드에는 붙여야 한다
  - 해당 객체의 논리적 상태와 무관한 필드에 경우만 transient를 생략해도 된다

- 직렬화에서 동기화 메커니즘 사용
  ```java
  public synchronized void writeObject(ObjectOutputStream os) throws IOException {
      s.defaultWriteObject();
  }
  ```
  - 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다
  - 그렇지 않으면 자원-순서 교착상태(resource-ordering deadlock)에 빠질 수 있다

- serialVersionUID
  - 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여해야 한다
  - 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하면 안된다

##### Item88. readObject 메서드는 방어적으로 작성하라

- 

##### Item89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거타입을 사용하라

##### Item90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
