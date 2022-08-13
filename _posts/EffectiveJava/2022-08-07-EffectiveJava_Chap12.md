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

- 깨지기 쉬운 직렬화에서의 불변식
  - item 50에서는 불변인 날짜 범위 클래스를 만드는 데 가변 Date 필드를 이용
  - 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 코드가 길어졌다
  ```java
  public final class Period {
      private final Date start;
      private final Date end;

      public Period(Date start, Date end) {
          this.start = new Date(start.getTime());
          this.end = new Date(end.getTime());

          if (this.start.compareTo(this.end) > 0)
              throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
      }

      public Date start() {
          return new Date(start.getTime());
      }

      public Date end() {
          return new Date(end.getTime());
      }
  }
  ```
  - Period 객체의 물리적 표현이 논리적 표현과 부합하므로 직렬화하기 위해선 Serializable 인터페이스를 구현하기만 하면 될 것 같지만 그런 경우 불변식을 보장하지 못하게 된다.
    - 원인은 readObject 메서드에 있다
      - readObject 메서드는 실질적으로 또 다른 public 생성자라고 할 수 있다
    - 생성자가 수행하는 조건들을 readObject에도 똑같이 수행하지 않으면 공격자는 아주 손쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다

- 직렬화에서의 불변식 보완
  - readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다
  - 바이트 스트림은 보통 정상적으로 생성된 인스턴스를 직렬화해서 만들어지는 데 바이트 스트림을 의도적으로 수정하거나 생성하여 readObject에 건네면 문제가 생기게 된다
    - 정상적인 생성자로는 만들 수 없는 객체가 생성되기 때문
  - 이 문제를 해결하기 위해서는 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사가 필요
    - 이 유효성 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다
    
  ```java
  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
      s.defaultReadObject();

      // 불변식을 만족하는지 검사한다.
      if(start.compareTo(end) > 0) {
          throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
      }
  }
  ```
  - 하지만 정상적으로 직렬화된 Period 인스턴스의 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수 있다
  - 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후, 스트림 끝에 추가되어 있는 '악의적인 객체 참조'를 읽어 Period 객체의 내부 정보를 얻을 수 있다 
    - 이 참조로 얻은 Date 인스턴스들을 검사 없이 수정해버릴 수도 있으니, Period 인스턴스의 필드는 더 이상 검사되지 않는다.
  - 공격 예시
  ```java
  public class MutablePeriod {
      //Period 인스턴스
      public final Period period;

      //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
      public final Date start;
      //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
      public final Date end;

      public MutablePeriod() {
          try {
              ByteArrayOutputStream bos = new ByteArrayOutputStream();
              ObjectArrayOutputStream out = new ObjectArrayOutputStream(bos);

              //유효한 Period 인스턴스를 직렬화한다.
              out.writeObject(new Period(new Date(), new Date()));

              /**
               * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
               * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
               */
              byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
              bos.write(ref); // 시작 start 필드 참조 추가
              ref[4] = 4; //참조 #4
              bos.write(ref); // 종료(end) 필드 참조 추가

              // Period 역직렬화 후 Date 참조를 훔친다.
              ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
              period = (Period) in.readObject();
              start = (Date) in.readObject();
              end = (Date) in.readObject();
          } catch (IOException | ClassNotFoundException e) {
              throw new AssertionError(e);
          }
      }
  }
  ```
  ```java
  public static void main(String[] args) {
      MutablePeriod mp = new MutablePeriod();
      Period p = mp.period;
      Date pEnd = mp.end;

      //시간 되돌리기
      pEnd.setYear(78);
      System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978

      //60년대로 회귀
      pEnd.setYear(60);
      System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
  }
  ```
    - 이 예에서 Period 인스턴스는 불변식을 유지한 채 생성됐지만, 이렇게 의도적으로 내부의 값을 수정할 수 있다
    - 이처럼 변경할 수 있는 Period 인스턴스를 획득한 공격자는 이 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청난 보안 문제를 일으킬 수 있다
      - 원인은 Period의 readObject()가 방어적 복사를 충분히 하지 않은 데 있다.
    - 객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는, 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.
      - readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.
    - 해결 방법
    ```java
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();

        // 가변 요소들을 방어적으로 복사한다.
        start = new Date(start.getTime());
        end = new Date(end.getTime());

        // 불변식을 만족하는지 검사한다.
        if(start.compareTo(end) > 0) {
            throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
        }
    }
    ```
      - 방어적 복사를 유효성 검사보다 앞서 수행
      - 만약 유효성 검사가 방어적 복사보다 앞에 있다면, 유효성 검사를 통과한 후 방어적으로 복사하기 전에 공격자가 참조를 통해서 Date값을 바꿔버리고 그 후에 방어적으로 복사하게 되기 때문
      - final 필드는 방어적 복사가 불가능하기 때문에, start와 end 필드에서 final 한정자를 제거 필요

- 기본 readObject를 사용해도 되는 경우
  - transient 필드를 제외한 모든 필드의 값을 매개변수로 받아, 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮다면 기본 readObject를 사용해도 된다.
  - 그렇지 않다면 커스텀 readObject()를 만들어 생성자에서의 유효성 검사와 동일한 수준의 검사를 해야 한다. 그리고 방어적 복사는 필수이다. 

##### Item89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거타입을 사용하라

- 싱글턴으로 구현된 클래스는 인스턴스를 하나만 만들어지는 것을 보장한다.

- 클래스에 implements Serializable 을 추가하는 순간 더 이상 싱글턴이 아니게 된다
  - 기본 직렬화를 쓰지 않더라도(아이템 87), 그리고 명시적인 readObject를 제공하더라도(아이템 88) 소용없다
  - 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다

  - readResolve 기능을 이용하는 경우
    - readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다
    - 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해뒀다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다
    - 대부분의 경우 이때 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다.
  
  - Serializable의 구현과 readResolve 메서드 제공
    - 싱글턴의 속성을 유지하기 위한 방법: readResolve()
      - 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 인스턴스를 반환
      - 이때 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없기 때문에 모든 인스턴스 필드를 transient로 선언
    - readResolve()를 인스턴스 통제 목적으로 사용하려는 경우
      - 객체 참조 타입 인스턴스 필드는 모두 transient로 선언
      - 위 조건이 충족되지 않은 경우 아이템 88 내용처럼 MutablePeriod 공격과 비슷한 방식으로 readResolve() 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다

- 역직렬화 공격방식
  - 싱글턴이 Transient가 아닌 참조 필드를 가지고 있다면, 그 필드 내용은 readResolve()가 실행되기 전에 역직렬화 된다
  - 그렇다면 잘 조작된 스트림을 써서 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다
  - 역직렬화 공격방식
    - readResolve 메서드와 인스턴스 필드 하나를 포함한 도둑(stealer) 클래스를 작성한다
    - 해당 클래스의 인스턴스 필드는 도둑이 숨길 직렬화된 싱글턴을 참조하는 역할을 한다
    - 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체한다
    - 이제 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하는 순환고리가 만들어졌다
    - 싱글턴이 도둑을 포함하므로 싱글턴이 역질렬화될 때 도둑의 readResolve 메서드가 먼저 호출된다
    - 그 결과, 도둑의 readResolve 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인(그리고 readResolve가 수행되기 전인) 싱글턴의 참조가 담겨 있게 된다
    - 도둑의 readResolve 메서드는 이 인스턴스 필드가 팜조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다.
    - 그런 다음 이 메서드는 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다
    - 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 ClassCastException을 던진다

  - 잘못된 싱글턴 예시
    - transient가 아닌 참조 필드를 가지고 있는 경우
    ```java
    public class Elvis implements Serializable {
        public static final Elvis INSTANCE = newElvis();

        private Elvis() {
        }

        private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

        public void printFavorites() {
            System.out.println(Arrays.toString(favoriteSongs));
        }

        private Object readResolve() {
            return INSTANCE;
        }
    }
    ```
    - non-transient 참조 필드를 훔쳐오는 도둑(stealer) 클래스
    ```java
    public class ElvisStealer implements Serializable {
        static Elvis impersonator;
        private Elvis payload;

        private Object readResolve() {
            // resolve되기 전의 Elvis 인스턴스의 참조를 저장한다. 
            impersonator = payload;
            // favoriteSongs 필드에 맞는 타입의 객체를 반환한다. 
            return new String[]{"A Fool Such as I"};
        }

        private static final long serialVersionUID = 0;
    }
    public class ElvisImpersonator {
        // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림
        private static final byte[] serializedForm = {
                -84, -19, 0, 5, 115, 114, 0, 20, 107, 114, 46, 115,
                101, 111, 107, 46, 105, 116, 101, 109, 56, 57, 46, 69,
                108, 118, 105, 115, 98, -14, -118, -33, -113, -3, -32,
                70, 2, 0, 1, 91, 0, 13, 102, 97, 118, 111, 114, 105, 116,
                101, 83, 111, 110, 103, 115, 116, 0, 19, 91, 76, 106, 97,
                118, 97, 47, 108, 97, 110, 103, 47, 83, 116, 114, 105, 110,
                103, 59, 120, 112, 117, 114, 0, 19, 91, 76, 106, 97, 118,
                97, 46, 108, 97, 110, 103, 46, 83, 116, 114, 105, 110, 103,
                59, -83, -46, 86, -25, -23, 29, 123, 71, 2, 0, 0, 120, 112,
                0, 0, 0, 2, 116, 0, 9, 72, 111, 117, 110, 100, 32, 68, 111,
                103, 116, 0, 16, 72, 101, 97, 114, 116, 98, 114, 101, 97, 107,
                32, 72, 111, 116, 101, 108
        };

        public static void main(String[] args) {
            // ElvisStealer.impersonator를 초기화한 다음,
            // 진짜 Elvis(즉 Elvis.INSTANCE)를 반환한다.
            Elvis elvis = (Elvis) deserialize(serializedForm);
            Elvis impersonator = ElvisStealer.impersonator;

            elvis.printFavorites();
            impersonator.printFavorites();
        }
    }
    ```
  - 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성한다
    - favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있지만, 해당 클래스를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 낫다
    - 도둑 클래스 공격으로 보여줬듯이 readResolve 메서드를 사용해 '순간적으로' 만들어진 역질렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이 써야 하는 작업

- Singleton을 보장하는 열거 타입 클래스
  - 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다
  - 공격자가 AccessibleObject.setAccessible 같은 privileged 메서드를 악용하는 경우, 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 모든 방어가 무력화된다
  ```java
  public enum Elvis {
      INSTANCE;
      private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

      public void printFavorites() {
          System.out.printtn(Arrays.toString(favoriteSongs));
      }
  }
  ```
  - 인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다
  - 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일 타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문

- readResolve 메서드의 접근성에 대한 이야기
  - final 클래스인경우 readResolve 메서드는 private 접근 제한자 이어야 한다
  - final 이 아닌 클래스의 경우 주의사항
    - 접근 제한자 설정시 private으로 선언하면 하위 클래스에서 사용할 수 없다
    - package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다
    - protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다
    - protected나 public이면서 하위 클래스에서 재정의 하지않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다

##### Item90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

- 안정적으로 역직렬화를 할 수 있는 직렬화 프록시 패턴이 있다.
```java
  class Period implements Serializable {
      private final Date start;
      private final Date end;

      public Period(Date start, Date end) {
          this.start = start;
          this.end = end;
      }

      private static class SerializationProxy implements Serializable {
          private static final long serialVersionUID = 123456789L;
          private final Date start;
          private final Date end;

          public SerializationProxy(Period p) {
              this.start = p.start;
              this.end = p.end;
          }

          private Object readResolve() {
              return new Period(start, end);
          }
      }

      private Object writeReplace() {
          return new SerializationProxy(this);
      }

      // 바깥 클래스의 직렬화 인스턴스 생성 불가
      private void readObject(ObjectInputStream stream) throws InvalidObjectException {
          throw new InvalidObjectException("프록시가 필요합니다.");
      }
  }
  
  - 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다
  - 중첩 클래스의 생성자는 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다
  - 중첩 클래스의 생성자는 단순히 인수로 넘어온 데이터를 복사한다
  - 직렬화를 수행하는 경우 프록시의 인스턴스를 반환한다

  - 직렬화 프록시 패턴 장점
    - 직렬화의 특성을 무시하고, 일반 인스턴스를 만들 때와 비슷하게 역직렬화된 인스턴스를 생성한다
    - 역직렬화된 인스턴스가 불변식을 검사하지 않아도 된다
    - 가짜 바이트 스트림 공격, 내부 필드 탈취 공격을 프록시 수준에서 차단할 수 있다
    - 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다
  
  - 직렬화 프록시 패턴 단점
    - 클라이언트가 확장할 수 있는 클래스에는 적용할 수 없다
    - 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다
    - 참조하고 있는 객체의 메서드를 프록시의 readResolve() 메서드 안에서 호출하는 경우 실제 객체가 생성된 것이 아니기 때문에 ClassCastException이 발생할 수 있다
    - 속도가 다소 느리다
