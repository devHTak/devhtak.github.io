---
layout: post
title: Querydsl 환경설정
summary: Querydsl
author: devhtak
date: '2021-09-19 21:41:00 +0900'
category: JPA
---

#### Querydsl 환경설정

- Gradle
  ```
  plugins {
      id 'org.springframework.boot' version ‘2.2.2.RELEASE'
      id 'io.spring.dependency-management' version '1.0.8.RELEASE'
      id 'java'
  }
  group = 'study'
  version = '0.0.1-SNAPSHOT'
  sourceCompatibility = '1.8'
  configurations {
      compileOnly {
          extendsFrom annotationProcessor
      }
  }
  repositories {
      mavenCentral()
  }
  dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
      implementation 'org.springframework.boot:spring-boot-starter-web'
      compileOnly 'org.projectlombok:lombok'
      runtimeOnly 'com.h2database:h2'
      annotationProcessor 'org.projectlombok:lombok'
      testImplementation('org.springframework.boot:spring-boot-starter-test') {
          exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
      }
  }
  test {
      useJUnitPlatform()
  }
  ```
  
- Maven
  - dependency
    ```
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
    </dependency>
    ```
  - plugin
    ```
    <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>apt-maven-plugin</artifactId>
        <executions>
            <execution>
                <goals>
                    <goal>process</goal>
                </goals>
                <configuration>
                    <outputDirectory>target/generated-sources/java</outputDirectory>
                    <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                </configuration>
            </execution>
        </executions>
    </plugin>
    ```
- Entity를 생성하면 /target/generated-sources/java 하위에 QEntity 파일이 생성된다.
- application.yml 로 JPA 설정
  ```
  spring:
    datasource:
      url: jdbc:h2:tcp://localhost/~/querydsl
      username: sa
      password:
      driver-class-name: org.h2.Driver
    jpa:
      hibernate:
        ddl-auto: create
      properties:
        hibernate:
          # show_sql: true
          format_sql: true
  logging.level:
    org.hibernate.SQL: debug
    # org.hibernate.type: trace
  ```
  - show_sql : 옵션은 System.out 에 하이버네이트 실행 SQL을 남긴다.
  - org.hibernate.SQL : 옵션은 logger를 통해 하이버네이트 실행 SQL을 남긴다.

#### 예제

- Hello.java
  ```java
  @Entity
  @Getter @Setter
  public class Hello {
        @Id @GeneratedValue
        private Long id;
  }
  ```
  
- QHello.java
  ```java
  @Generated("com.querydsl.codegen.EntitySerializer")
  public class QHello extends EntityPathBase<Hello> {

        private static final long serialVersionUID = -363860077L;

        public static final QHello hello = new QHello("hello");

        public final NumberPath<Long> id = createNumber("id", Long.class);

        public QHello(String variable) {
            super(Hello.class, forVariable(variable));
        }

        public QHello(Path<? extends Hello> path) {
            super(path.getType(), path.getMetadata());
        }

        public QHello(PathMetadata metadata) {
            super(Hello.class, metadata);
        }

  }
  ```
  - QHello 는 Querydsl이 직접 생성해준다.
  
- Test
  ```java
  @SpringBootTest
  @Transactional
  class QuerydslApplicationTests {

      @PersistenceContext
      EntityManager entityManager;

      @Test
      void contextLoads() {
          Hello hello = new Hello();
          entityManager.persist(hello);

          JPAQueryFactory query = new JPAQueryFactory(entityManager);
          QHello qHello = new QHello("h"); // QHello.hello; 가능

          Hello result = query
              .selectFrom(qHello)
              .fetchOne();

          assertThat(result).isEqualTo(hello);
          assertThat(result.getId()).isEqualTo(hello.getId());
      }
  }
  ```
  
####  출처

- 실전! Querydsl \[인프런 김영한님 강의]
