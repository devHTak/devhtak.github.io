---
layout: post
title: \[더 자바, 코드를 조작하는 다양한 방법\] 바이트코드
summary: The Java
author: devhtak
date: '2021-01-17 21:41:00 +0900'
category: Java
---

#### 코드 커버리지는 어떻게 측정할까?

- 바이트코드를 사용한 대표적인 예제
- 코드 커버리지? 테스트 코드가 확인한 소스 코드를 %로 나타낸다.
  - JaCoCo 사용
  - (JaCoCo) https://www.eclemma.org/jacoco/trunk/doc/index.html
  - (Test Coverage 논문) http://www.semdesigns.com/Company/Publications/TestCoverage.pdf
  - 사용방법
    - pom.xml에 플러그인 추가 후 maven 업데이트
      ```
      <plugin>
          <groupId>org.jacoco</groupId>
          <artifactId>jacoco-maven-plugin</artifactId>
          <version>0.8.4</version>
          <executions>
              <execution>
                  <goals>
                    <goal>prepare-agent</goal>
                  </goals>
              </execution>
              <execution>
                  <id>report</id>
                  <phase>prepare-package</phase>
                  <goals>
                      <goal>report</goal>
                  </goals>
              </execution>
          </executions>
      </plugin>
      ```
    - 메이븐 verify
      ```
      $ mvn clean verify
      ```
    - target\site\jacoco\index.html 파일을 확인하면 패당 패키지에 클래스들의 coverage를 확인할 수 있다.
    
    - 커버리지가 낮은 경우 빌드가 되지 않게 설정할 수 있다.
      ```
      <execution>
          <id>jacoco-check</id>
          <goals>
              <goal>check</goal>
          </goals>
          <configuration>
              <rules>
                  <rule>
                      <element>PACKAGE</element>
                      <limits>
                          <limit>
                              <counter>LINE</counter>
                              <value>COVEREDRATIO</value>
                              <minimum>0.50</minimum>
                          </limit>
                      </limits>
                  </rule>
              </rules>
          </configuration>
      </execution>
      ```
    
  - 어떻게 코드 커버리지를 파악할까?
    - 가장 간단한 방법: 코드 커버리지에서 파악하여 라인 수를 카운팅과 코드 실행할 때 지나각 코드 라인 수를 카운팅하여 비교
  
#### 바이트 코드 조작

- 바이트 코드 조작 라이브러리
  - ASM
  - Javassist
  - ByteBuddy

- 예제: ByteBuddy 사용
  - 아무것도 없는 Moja에서 "Rabbit"을 꺼내는 마술
    ```java
    public class Moja {
        public String pullOut() {
            return "";
        }
    }
    ```
    ```java
    public class Magician {
        public static void main(String[] args) {
            System.out.println(new Moja().pullOut());
        }
    }
    ```
  - 바이트버디 사용
    ```java
    public class Magician {
        public static void main(String[] args) {
            try {
                new ByteBuddy().redefine(Moja.class)
                    .method(named("pullOut")).intercept(FixedValue.value("Rabbit"))
                    .make().saveIn(new File("/Users/workspace/HalleHomework/target/classes/"));
            } catch(IOException e) {
                e.printStackTrace();
            }
            
            System.out.println(new Moja().pullOut()); // Rabbit 출력
        }
    }
    ```
    - Moja.java 파일안에 pullOut 메소드는 ""를 리턴한다.
    - Moja.class 파일안에 pullOut 메소드는 "Rabbit"을 리턴한다.
    - 수정하여 다시 컴파일 하기 전까지는 Moja.class 파일 안에 pullOut 메소드는 "Rabbit"을 리턴한다.

- 예제) Javaagent





