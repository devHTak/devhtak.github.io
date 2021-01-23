---
layout: post
title: 더 자바, 코드를 조작하는 다양한 방법_바이트코드
summary: The Java
author: devhtak
date: '2021-01-17 21:41:00 +0900'
category: The Java
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
    - 만약 main 메소드에 try문이 적용된 Moja.class를 갖고 싶다면
      - main 메소드를 2번 실행한다.
      - 1번째 실행할 때에는 컴파일 하듯 Moja.java 파일안에 pullOut 메소드는 ""를 리턴하는 클래스 파일은 "Rabbit"을 리턴하는 클래스 파일로 다시 만든다. 
        - 아직은 ""를 리턴하는 클래스파일이기 때문에, ""이 출력
        - main 메소드가 실행되기 전에 클래스로더에 의해 Moja.class가 메모리에 적재되고 해당 Moja.class는 조작되기 이전이다.
      - 2번째 실행으로 "Rabbit" 클래스파일이 존재하기 때문에 "Rabbit"이 출력된다.
      - 1번째 실행했을 때 바이트 코드를 조작하고, 2번째 실행했을 때 조작된 바이트코드를 실행하는 것
    - Moja.java 파일안에 pullOut 메소드는 ""를 리턴한다.
    - Moja.class 파일안에 pullOut 메소드는 "Rabbit"을 리턴한다.
    - 수정하여 다시 컴파일 하기 전까지는 Moja.class 파일 안에 pullOut 메소드는 "Rabbit"을 리턴한다.
    
- Javaagent 활용
  - main 메소드를 실행하기 전에 javaagent에 prev-main으로 지정한 메소드를 실행할 수 있다.
  - class 파일을 변경하는 것이 아닌 클래스 로더에서 로딩할 때 변경된 바이트코드가 메모리에 적재되도록 한다.
    - Transparent
  
  - 예제
    - agent 프로젝트 생성 후 ByteBuddy 의존성 추가
      ```xml
      <dependency>
          <groupId>net.bytebuddy</groupId>
          <artifactId>byte-buddy</artifactId>
          <version>1.10.8</version>
      </dependency>
      ```
      
    - agent 프로젝트에 premain 구현
      ```java
      public class MagicianAgent {
          public static void premain(String agentArgs, Instrumentation inst){
              new AgentBuilder.Default()
                .type(ElementMatchers.any())
                .transform(new AgentBuilder.Transformer(){
                    @Override
                    public DynamicType.Builder<?> transform(DynamicType.Builder<?> builder, TypeDescription typeDescription, ClassLoader classLoader, JavaModule javaModule) {
                            return builder.method(named("pullOut")).intercept(FixedValue.value("Rabbit"));
                    }
                }).installOn(inst);
          }
      }
      ```
    
    - agent 프로젝트 jar 패키징
      - plugin 설정
        ```xml
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <archive>
                    <index>true</index>
                    <manifest>
                        <addClasspath>true</addClasspath>
                    </manifest>
                  <manifestEntries>
                      <mode>development</mode>
                      <url>${project.url}</url>
                      <key>value</key>
                      <!-- pre-main으로 사용할 클래스 설정 -->
                      <Premain-Class>com.study.MagicianAgent</Premain-Class>
                      <!-- class 재정의 가능여부 설정 -->
                      <Can-Redefine-Classes>true</Can-Redefine-Classes>
                      <Can-Retransform-Classes>true</Can-Retransform-Classes>
                  </manifestEntries>
                </archive>
            </configuration>
        </plugin>
        ```
        - 참고 자료: Apache Maven JAR Plugin(https://maven.apache.org/plugins/maven-jar-plugin/examples/manifest-customization.html)
        - 참고 자료: java.lang.instrument(https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)
        
      - 패키지
        ```
        mvn clean pakaging
        ```
    
    - 클래스 조작 패키지에서 javaagent 적용
      - VM Option에 javaagent jar 설정
        ```
        -javaagent:/Users/workspace/magicianAgent/target/magicianAgent-1.0-SBAOSHOT.jar
        ```
    

#### 바이트코드 조작 툴 활용 예시

- 프로그램 분석
  - 코드에서 버그 찾는 툴
  - 코드 복잡도 계산
  
- 클래스 파일 생성
  - 프록시
    - Test에서 Mock은 프록시로 만든다.
    - JPA에서 FetchType.LAZY를 하면 해당 객체를 프록시로 만든다. 
  - 특정 API 호출 접근 제한
  - 스칼라 같은 언어의 컴파일러
  
- 스프링이 컴포넌트 스캔을 하는 방법 (asm)
  - 컴포넌트 스캔으로 빈으로 등록할 후보 클래스 정보를 찾는데 사용
  - ClassPathScanningCandidateComponentProvider -> SimpleMetadataReader
  - ClassReader와 Visitor 사용해서 클래스에 있는 메타 정보를 읽어온다.



