---
layout: post
title: 스터디 할래 7. 패키지
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-25 21:41:00 +0900'
category: Java Study
---

#### 학습할 것

- package 키워드
- import 키워드
- 클래스패스
- CLASSPATH 환경변수
- -classpath 옵션
- 접근지시자

#### package 키워드

- 패키지
  - 클래스를 구분짓는 폴더 개념
  - 자바는 패키지의 가장 상위 디렉터리(root)에서 실행해야 한다는 약속이 있기 때문에 해당 패키지로 가서 컴파일하지 않는다.
  - 소스에 가장 첫 줄에 있어야 하고, 패키지 선언은 소스 하나에 하나만 있어야 한다.
  - 패키지 이름과 위치한 디렉터리의 이름이 같아야 한다.
  - 패키지 이름을 java로 시작하면 안된다.
  - 모든 클래스에는 정의도니 클래스 이름과 패키지 이름이 있다.
    - FQCN(Full Qulified Class Name): 패키지 이름 + 클래스 이름을 합쳐 완전하게 한 클래스로 표현할 수 있어야 한다.
    
- 빌트-인 패키기(built-in package)
  - java.lang, java.util 패키지는 import하지 않아도 알아서 해당 패키지의 클래스를 불러와 사용할 수 있다.

#### import 키워드

- 다른 패키지명에 있는 클래스를 찾지 못할 때 사용한다.
- 빌트-인 패키지는 import하지 않아도 사용이 가능하다.
- import static
  - static한 변수, 메소드를 사용하고자 할 때 용이하다.
  - import static을 사용하지 않으면 클래스.클래스변수 또는 클래스.메소드 형식으로 사용해야 하지만 해당 변수, 메소드만 import하여 사용할 수 있다.

#### 클래스패스

- 클래스를 찾기위한 경로
- JVM이 프로그램을 실행할 때 클래스파일을 찾는데 클래스파일(기준이 되는 경로)을 사용한다.
- .java를 컴파일하면 바이트 코드로 변환된 .class 파일이 생성된다. java runtime으로 이 .class 파일에 포함된 명령을 실행하면 해당 파일을 찾을 수 있어야 한다.
- .class파일을 찾을 때, classpath에 지정된 경로를 사용한다.
- classpath 지정 방법( ;으로 여러 디렉터리를 구분하여 줄 수 있다.)
  - CLASSPATH 환경변수 사용
  - java runtime에서 -classpath 옵션 사용

#### CLASSPATH 환경변수

- 컴퓨터 시스템 변수 설정을 통해 지정할 수 있다.
- JVM이 시작될 때 JVM의 클래스 로더는 CLASSPATH 환경 변수를 호출한다.
- CLASSPATH 환경변수 지정 방법
  - https://hyoje420.tistory.com/7

#### -classpath 옵션

- 컴파일러가 컴파일하기 위해서 필요로 하는 참조할 클래스 파일들을 찾기 위해서 컴파일 시 파일 경로를 지정해주는 옵션
```
javac <option> <source files>
```
- 옵션에 -classpath 또는 -cp 를 주어 설정할 수 있다.


#### 접근지시자

- 접근지시자는 클래스, 메소드, 인스턴스 및 클래스 변수를 선언할 때 사용한다.
- 객체지향에서 정보은닉, 캡슐화에 사용된다.

|-|동일 클래스|동일 패키지|상속 관계|그 외|
|---|---|---|---|---|
|public|O|O|O|O|
|protected|O|O|O|X|
|default|O|O|X|X|
|private|O|X|X|X|
