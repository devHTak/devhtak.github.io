---
layout: post
title: 스터디 할래 13. I/O
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-02-23 21:41:00 +0900'
category: Java Study
---

#### 학습할 것
- 스트림 (Stream) / 버퍼 (Buffer) / 채널 (Channel) 기반의 I/O
- InputStream과 OutputStream
- Byte와 Character 스트림
- 표준 스트림 (System.in, System.out, System.err)
- 파일 읽고 쓰기

#### 스트림 (Stream) / 버퍼 (Buffer) / 채널 (Channel) 기반의 I/O

- 스트림
  - FIFO(First In First Out)
  - 단방향이기 떄문에 입력 스트림과 출력스트림을 별도로 사용해야 한다.
  - 연속된 데이터의 흐름으로 입출력 진행 시 다른 작업을 할 수 없는 Blocking 상태가 된다.
  - 입출력 대상을 변경하기 편하며 동일한 프로그램 구조를 유지할 수 있다.
  
  - 데이터를 전송하는 방식에 따라 2가지로 나뉜다.
    - 바이트 스트림
      - 데이터를 Byte 단위(8 bit)로 주고 받는 것을 의미
      - binary 데이터를 입력하는 스트림
      - 이미지, 동영상 등 모든 종류의 데이터들을 송수신할 때 주로 사용
      - InputStream, OutputStream이 바이트 스트림 최고 조상
    - 문자 스트림(Character Stream)
      - 문자 단위로 인코딩 처리하는 스트림
      - 텍스트 파일등을 송수신할 때 사용

- 버퍼
  - 버퍼란 데이터를 임시 저장하는 공간을 의미한다.
  - 스트림에서 입, 출력이 바로 전달되어 I/O의 횟수가 늘어나지만, 버퍼를 사용하면 버퍼가 가득차거나, 개행문자가 나타날 때 한번에 전송하기 떄문에 I/O 횟수가 줄어 성능상 이점이 있다.
  - 어떤 메모리(위치)를 사용하느냐에 따라 Non-Direct와 Direct로 나뉜다.
  
    |구분|Non-Direct|Direct|
    |---|---|---|
    |메모리 공간 위치|힙 영역|운영체제(OS)|
    |버퍼가 생성되는 시간|빠름|느림|
    |버퍼 크기|작음|큼 (큰 데이터 처리에 유리)|
    |입출력 성능|낮음|높음 (입출력이 빈번할 떄 유리)|   

- 자바 NIO(New I/O)
  - byte, char, int 등 기본 데이터 타입을 저장할 수 있는 저장소로서, 배열과 마찬가지로 제한된 크기(capacity)에 순서대로 데이터를 지정한다.
  - 버퍼는 데이터를 저장하기 위한 것이지만, 실제로 버퍼가 사용되는 것은 채널을 통해서 데이터를 주고 받을 때 쓰인다.
  - 채널을 통해서 소켓, 파일 등에 데이터를 전송할 때나 읽어올 때 버퍼를 사용하게 됨으로써 가비지량을 최소화 시킬 수 있게 되며, 이는 가비지 콜렉션 회수를 줄임으로써 서버의 전체 처리량을 증가시킨다.

- 채널
  - 데이터가 통과하는 쌍방향 통로로, 채널에서 데이터를 주고 받을 때 사용되는 것이 버퍼
  - 채널에서 소켓과 연결된 SocketChannel, 파일과 연결된 FileChannel, 파이프와 연결된 Pipe.SinkChannel과 Pipe.SourceChannel 등이 존재하며, 서버 소켓과 연결된 ServerScoketChannel도 존재 

- IO vs NIO
  - IO 방식으로 각각의 스트림에서 read() 와 write() 가 호출이 되면 데이터가 입력 되고, 데이터가 출력되기전까지, 스레드는 블로킹(멈춤) 상태가 된다. 
  - 이렇게 되면 작업이 끝날때까지 기다려야 하며, 그 이전에는 해당 IO 스레드는 사용할 수 없게 되고, 인터럽트도 할 수 없다. 블로킹을 빠져나오려면 스트림을 닫는 방법 밖에 없다.
  - NIO 의 블로킹 상태에서는 Interrupt 를 이용하여 빠져나올 수 있다.

  |구분|IO|NIO|
  |---|---|---|
  |입출력 방식|스트림|채널|
  |버퍼 방식|Non-Buffer|Buffer|
  |비동기 방식 지원|X|O|
  |Blocking/Non-Blocking 방식|Blocking Only|Both|
  |사용 케이스|연결 클라이언트가 적고, IO 가 큰 경우(대용량)|연결 클라이언트가 많고, IO 처리가 작은 경우(저용량)|
  
  - Blocking
    - 프로그램 제어권에 대한 단어
    - 호출자에서 메서드를 호출했을 때 끝날 때까지 프로그램의 제어권을 호출자에게 반납하지 않는 것을 말한다.
    - 따라서 작업하는 메서드 이외에는 호출하는 쪽의 동작이 멈추고 메서드의 실행이 끝날때까지 기다린다.
    - 하나의 호출마다 하나의 쓰레드가 필요하다.

  - Non-Blocking
    - 호출받는 메서드가 프로그램의 제어권은 바로 반납하고, 호출자 쪽은 호출하는 대로, 메서드는 메서드대로 멀티쓰레드 같은 방식으로 동작
    - 요청을 하는 쓰레드 하나만 있으면 된다.

#### InputStream과 OutputStream

- InputStream
  - 바이트 기반 입력 스트림의 최상위 추상 클래스
  - 모든 바이트 기반 입력 스트림은 이 클래스를 상속 받아서 만들어 진다.
  - 버퍼, 파일, 네트워크 단에서 입력되는 데이터를 읽어오는 기능을 수행한다.

  |메서드|설명|
  |---|---|
  |read()|입력 스트림으로부터 1바이트를 읽어서 바이트를 리턴|
  |read(byte[] b)|입력 스트림으로부터 읽은 바이트들을 매개값으로 주어진 바이트 배열 b 에 저장하고 실제로 읽은 바이트 수를 리턴|
  |read(byte[] b, int off, int len)|입력 스트림으로부터 len 개의 바이트만큼 읽고 매개값으로 주어진 바이트 배열 b\[off] 부터 len 개까지 저장. 그리고 실제로 읽은 바이트 수인 len 개를 리턴. 만약 len 개를 모두 읽지 못하면 실제로 읽은 바이트 수를 리턴|
  |close()|사용한 시스템 자원을 반납하고 입력 스트림 닫기|
  
- OutputStream
  - 바이트 기반 출력 스트림의 최상위 추상 클래스
  - 모든 바이트 기반 출력 스트림은 이 클래스를 상속 받아서 만들어 진다.
  - 버퍼, 파일, 네트워크 단으로 데이터를 내보내는 기능을 수행한다.

  |메서드|설명|
  |---|---|
  |write(int b)|출력 스트림으로부터 1바이트를 보낸다.(b 의 끝 1바이트|
  |write(byte[] b)|출력 스트림으로부터 주어진 바이트 배열 b의 모든 바이트를 보낸다.|
  |write(byte[ ] b, int off, int len)|출력 스트림으로 주어진 바이트 배열 b[off] 부터 len 개까지의 바이트를 보낸다.|
  |flush()|버퍼에 잔류하는 모든 바이트를 출력한다.|
  |close()|사용한 시스템 자원을 반납하고 입력 스트림 닫기|

#### Byte와 Character 스트림

- Byte Stream
  - binary 데이터를 입출력하는 스트림
  - 데이터는 1바이트 단위로 처리
  - 이미지, 동영상 등을 송수신 할 때 주로 사용 
  
  - InputStream
    - ObjectInputStream
    - FileInputStream
    - PipedInputStream
    - FileInputStream
      - DataInputStream
      - BufferdInputStream
      - PushbackInputStream
      
  - OutputStream
    - ObjectOutputStream
    - FileOutputStream
    - PipedOutputStream
    - FilterOutputStream
      - DataOutputStream
      - BUfferedOutputStream

- Character Stream
  - text 데이터를 입출력하는 스트림
  - 데이터는 2바이트 단위로 처리
  - 일반적인 텍스트 및 JSON, HTML 등을 송수신할 때 주로 사용
  
  - Reader
    - BufferedReader
    - FilterReader
    - PipedReader
    - InputStreamReader
      - FileReader
      
  - Writer
    - BufferedWriter
    - FilterWriter
    - PipedWriter
    - PrintWriter
    - OutputStreamWriter
      - FileWriter

- 보조 스트림
  - FilterInputStream 과 FilterOutputStream 을 상속받는 클래스들로 기본 스트림과 결합하여 특정 상황에서 보다 편리하게 사용할 수 있다.
  - BufferedInputStream/BufferedOutputStream
    - 버퍼를 사용해 입출력 효율과 편의를 위해 사용
  - BufferedReader/BufferedWriter
    - 라인단위의 입출력이 편리함
  - InputStreamReader/OutputStreamReader
    - 바이트 스트림을 문자 스트림처럼 쓸 수 있도록하며 문자 인코딩 변환을 지원
  - DataInputStream/DataOutputStream
    - 자바 원시자료형 데이터 처리에 적합  

#### 표준 스트림 (System.in, System.out, System.err)

- 표준 입출력 스트림의 종류는 java.lang 패키지의 System 클래스 내부에 static 으로 선언되어 있으며 다음과 같다.
  ```java
  public final class System {
      public static final InputStream in;
      public static final PrintStream out;
      public static final PrintStream err;
      // ....
  }  
  ```

  - System.out 은 콘솔 화면에 문자열을 출력하기 위한 용도로 사용되는 출력 스트림이다.
  - System.in 은 키보드의 입력을 받아들이기 위해서 사용되는 입력 스트림이다.
  - System.out 과 System.err
    - 둘다 출력 스트림이다.
    - err 은 버퍼링을 지원하지 않는다. 
    - 이것은 err 이 보다 정확하고 빠르게 출력되어야 하기 때문이라고 한다. 
    - 버퍼링을 하던 도중 프로그램이 멈추면 버퍼링된 내용은 출력되지 않기 때문이다.

#### 파일 읽고 쓰기

- 텍스트 파일인 경우 문자 스트림 클래스들을 사용하면 되고, 바이너리 파일인 경우 바이트 스트림을 기본적으로 사용한다.
- 입출력 효율을 위해 Buffered 계열의 보조 스트림을 함께 사용하는 것이 좋다.

- 텍스트 파일인 경우
  ```java
  BufferedReader br = new BufferedReader(new FileReader("a.txt"));
  BufferedWriter bw = new BufferedWriter(new FileWriter("b.txt"));
  String s;
  while ((s = br.readLine()) != null) {
      bw.write(s + "\n");
  }
  ````

- 이진 파일인 경우
  ```java
  BufferedInputStream is = new BufferedInputStream(new FileInputStream("a.jpg"));
  BufferedOutputStream os = new BufferedOutputStream(new FileOutputStream("b.jpg"));
  byte[] buffer = new byte[16384];
  while (is.read(buffer) != -1) {
      os.write(buffer);
  }
  ```
  
#### 직렬화

- 객체를 데이터 스트림으로 만드는 것
- 객체에 저장된 데이터를 스트림에 쓰기위해 연속적인(serial) 데이터로 변환
- 반대로 스트림으로부터 데이터를 읽어서 객체를 만드는 것은 역직렬화(deserialization)이라고 한다.
- 직렬화 시 생성자 및 메소드는 직렬화에 포함되지 않고 필드만 변환된다.
  - static, transient가 붙은 필드는 직렬화되지 않는다.

- Serializable과 Transient
  - 직렬화가 가능한 클래스를 만드는 방법은 Serializable 인터페이스를 구현하는 것
    ```java
    public class TestInfo implements Serializable { ... }
    ```
  - Serializable을 구현한 클래스에 상속을 받는 자식 객체는 직렬화 대상이 된다.
  - Serializable을 구현한 클래스에 부모 클래스의 필드는 직렬화 대상애서 제외된다.
  - transient가 붙은 변수는 직렬화 대상에서 제외되며 인스턴스 변수의 값은 그 타입의 기본값으로 직렬화된다.
  - 

- ObjectInputStream과 ObjectOutputStream
  - 보조 스트림으로 입출력(직렬화/역직렬화) 스트림을 지정해야 한다.
    ```java
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("serializableTest.ser"));
    
    oos.writer(new TestInfo());
    ```
  - 파일에 객체를 저장(직렬화)하는 예제로 serializableTest에 TestInfo 객체를 직렬화하여 저장한다.
    ```java
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream("serializableTest.ser"));
    
    TestIno testInfo = (TestInfo)ois.readObject();
    ```
  - 파일에 저장한 객체를 가져와 읽어(역직렬화)한다.
    
- 직렬화 가능한 클래스의 버전 관리
  - 직렬화된 객체를 역직렬화할 때에는 직렬화했을 때와 같은 클래스를 사용해야 한다.
  - 클래스 이름이 같아도 클래스의 내용이 변경됐다면 역직렬화는 실패하고 에러가 발생한다.
  - 역직렬화할 때 serialVersionUID란 객체를 비교하여 버전 일치하는지 비교하는 데 작성하지 않으면 자동 생성된다.
  - 임의로 serialVersionUID를 생성하면 직렬화 -> 인스턴스 변수 추가 -> 역직렬화하더라도 에러없이 동작한다.
