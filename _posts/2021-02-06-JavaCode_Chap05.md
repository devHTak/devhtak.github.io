---
layout: post
title: 더 자바, 코드를 조작하는 다양한 방법_애노테이션프로세서
summary: The Java
author: devhtak
date: '2021-02-06 14:41:00 +0900'
category: The Java
---

#### lombok은 어떻게 동작할까

- lombok
  - @Getter, @Setter, @Builder 등의 애노테이션과 애노테이션 프로세서를 제공하여 표준적으로 작성해야 할 코드를 개발자 대신 생성해주는 라이브러리
  - lombok 사용하기
    ```
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.8</version>
        <scope>provided</scope>
    </dependency>
    ```
  - 동작 원리
    - 컴파일 시점에 애노테이션 프로세스를 사용하여 소스코드의 AST(Abstract Syntax Tree)를 조작한다.
      - https://javaparser.org/inspecting-an-ast/
    - 애노테이션 프로세서는 컴파일 단계에서 끼어들어서 특정한 Annotation이 붙어있는 소스코드를 참조하여 다른 소스를 만들어 내는 것을 말한다.
     
  - 논란 거리
    - 공개된 API가 아닌 컴파일러 내부 클래스를 사용하여 기존 소스 코드를 조작한다.
    - 특히 이클립스의 경우 java agent를 사용하여 컴파일러 클래스까지 조작하여 사용한다.
      - 해당 클래스들 역시 공개된 API가 아니다보니 버전 호환성 문제가 생길 수 있다.
    - 그럼에도 불구하고 편리성 때문에 널리 쓰이고 있으며 대안이 몇가지 있지만 롬복의 모든 기능과 편의성을 대체하지 못하는 현실
      - AutoValue (https://github.com/google/auto/blob/master/value/userguide/index.md)
      - Immutables (https://immutables.github.io)

- 참고
  - https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Processor.html
  - https://projectlombok.org/contributing/lombok-execution-path
  - https://stackoverflow.com/questions/36563807/can-i-add-a-method-to-a-class-from-a-compile-time-annotation
  - http://jnb.ociweb.com/jnb/jnbJan2010.html#controversy
  - https://www.oracle.com/technetwork/articles/grid/java-5-features-083037.html
  
#### Annotation Processor 1

- Processor Interface
  - https://docs.oracle.com/en/java/javase/11/docs/api/java.compiler/javax/annotation/processing/Processor.html
  - 여러 라운드(rounds)에 거쳐 소스 및 컴파일 된 코드를 처리할 수 있다.
  
- 예제
  - Moja project
    - Moja.class
      ```java
      @Magic
      public interface Moja {
          public void pullOut();
      }
      ```
      - Magic Annotation 활용
  
  - MagicMoja project
    - Magic Annotation
      ```java
      @Retention(RetentionPolicy.SOURCE)
      @Target(ElementType.TYPE) // interface, class, Enum
      public @interface Magic {

      }
      ```
      - Target에 TYPE을 하면 interface, class, enum에 사용할 수 있다.
      - 구체적이지 않기 때문에 Annotation Processor를 생성하여 interface에 붙는 경우 실패하도록 할 수 있다.
    
    - MaginProcess.class
      ```java
      @AutoService(Processor.class)
      public class MagicProcessor extends AbstractProcessor{
          @Override
          public Set<String> getSupportedAnnotationTypes() {
              // TODO Auto-generated method stub
              return Set.of(Magic.class.getName());
          }
          @Override
          public SourceVersion getSupportedSourceVersion() {
              // TODO Auto-generated method stub
              return SourceVersion.latestSupported();
          }
          @Override
          public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
              // TODO Auto-generated method stub
              Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Magic.class);
              for(Element element: elements) {
                  Name elementName = element.getSimpleName();
                  if(element.getKind() != ElementKind.INTERFACE) {
                      processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "Magic anntation can not be used on" + elementName);
                  } else {
                      processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "Processing " + elementName);
                  }
              }
              return true;
            }
      }
      ```
      - Process 인터페이스를 구현해도 되지만, 자바에서 제공하는 AbstractProcessor 추상클래스를 구현해도 된다.
      - Process 에서 구현해야하는 여러 메서드들을 구현해주고 있다.
      - getSupportAnnotationTypes() override
        - 이 프로세서가 처리할 애노테이션들을 지정
        - Element란? 
          - 패키지, 클래스, 메서드 등 소스코드의 구성요소를 Element라고 부른다. 
          - 각 Element 들이 프로세스를 할 때 참조할 수 있다.
      - getSupportedSourceVersion() override
        -  몇 버전의 소스코드를 지원하는지 설정.
      - boolean processor(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) override
        - 애노테이션 프로세서는 라운드 라는 개념으로 처리를 한다.
          - 여러 라운드에 거쳐 처리를 한다.
          - 각 라운드마다 프로세서에게 특정 애노테이션을 가지고 있는 엘리먼트를 찾으면 처리를 요청한다.
          - 처리된 결과가 다음 라운드에게 전달될 수 있다.
        - 만약 여기서 true를 리턴하면, 애노테이션 프로세서가 처리를 한것이다.
        - true를 리턴하면, 다른 프로세서가 이를 처리하지 않는다.
        - Round란?
          - 애노테이션 프로세서는 라운드라는 개념이 존재한다.
          - 여러 라운드에 걸쳐 처리를 한다.
          - 각 라운드 마다 애노테이션 프로세서가 처리할 엘리먼트를 찾으면, 이를 처리하고 그 결과를 다음 프로세서에게 넘길 수도 있다.
      - @AutoService
        - 자동적으로 MAINFEST파일을 생성해준다.
        - 컴파일 시점에 애노테이션 프로세서를 사용하여 META-INF/services/javax.annotation.processor.Processor 파일 자동으로 생성해 준다.
        
    - Packaging
      - mvn clean install
      - 생성된 jar파일을 Moja project의 pom.xml에 dependency 추가하여 사용
        ```
        <dependency>
            <groupId>com.study</groupId>
            <artifactId>MagicMoja</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        ```
        
- 유틸리티
  - AutoService: 서비스 프로바이더 레지스트리 생성기
    - https://github.com/google/auto/tree/master/service
    ```
    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
        <version>1.0-rc6</version>
    </dependency>
    ```

- Service Provider
  - https://itnext.io/java-service-provider-interface-understanding-it-via-code-30e1dd45a091

- 참고
  - http://hannesdorfmann.com/annotation-processing/annotationprocessing101
  - http://notatube.blogspot.com/2010/12/project-lombok-creating-custom.html
  - https://medium.com/@jintin/annotation-processing-in-java-3621cb05343a
  - https://medium.com/@iammert/annotation-processing-dont-repeat-yourself-generate-your-code-8425e60c6657
  - https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javac.html#processing
  
#### Annotation Processor 2부

- 예제
  - @Magic annotation을 붙이면 Moja의 구현체인 MagicMoja클래스를 생성
  - MagicProcessor.class
    ```java
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // TODO Auto-generated method stub
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Magic.class);
        for(Element element: elements) {
            Name elementName = element.getSimpleName();
            if(element.getKind() != ElementKind.INTERFACE) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "Magic anntation can not be used on" + elementName);
            } else {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "Processing " + elementName);
            }

            TypeElement typeElement = (TypeElement)element;
            ClassName className = ClassName.get(typeElement);

            // 원하는 메소드를 생성
            MethodSpec pullOut = MethodSpec.methodBuilder("pullOut")
                .addModifiers(Modifier.PUBLIC)
                .returns(String.class)
                .addStatement("return $S", "Rabbit!")
                .build();

            // 원하는 클래스 생성, pullOut 메소드를 넘겨준다
            TypeSpec MagicMoja = TypeSpec.classBuilder("MagicMoja")
                .addModifiers(Modifier.PUBLIC)
                .addMethod(pullOut)
                .addSuperinterface(className)
                .build();

            Filer filer = processingEnv.getFiler();
            try {
                JavaFile.builder(className.packageName(), MagicMoja)
                  .build().writeTo(filer);
            } catch(IOException e) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "Fatal error: " + e.getMessage());
            }
        }

        return true;
    }
    ```
    - ClassName
      - Element를 TypeElement로 변환한뒤, JavaPoet을 사용하여 ClassName 타입의 객체로 변환한다.
      - 이 타입의 객체는 클래스 정보들을 참조할 수 있다.
    - MethodSpec
      - 우리가 생성할 클래스의 메소드 스팩을 구현한다.
      - methodBuilder(메소드명): 구현할 메소드의 이름을 정의한다.
      - addModifireds(접근지시자를): 메소드의 접근 지시자를 정의한다.
      - returns: 해당 메소드에서 리턴하는 타입을 정의한다.
      - addStatement: 메소드 내부의 스테이트먼트를 정의한다.
    - TypeSpec
      - 우리가 생성한 클래스의 스팩을 구현한다.
      - classBuilder(클래스명): 구현할 클래스 명을 정의한다. (이때 풀패키지 경로가아닌 심플 네임만 지정해준다.)
      - addSuperInterface(인터페이스): 우리가 생성할 클래스가 구현할 인터페이스에 대한 정보를 정의한다.
      - addModifiers(접근지시자를): 클래스의 접근 지시자를 정의한다.
      - addMethod(메소드스팩): 클래스에 추가할 메소드 스팩을 정의한다. (위에서 정의한 pullOut 메소드를 추가한다.)
      - build 를 통해 클래스 스팩 구현을 마친다.
    - Filer
      - 소스코드, 클래스 코드 및 리소스를 생성할 수 있는 인터페이스이다.
      - JavaPoet를 사용한다면 보다 쉽게 생성이 가능하다.
      
  - Main 함수
    ```java
    Moja moja = new MagjcMoja();
		System.out.println(moja.pullOut());
    ```
    - MagicMoja를 생성하여 가져올 수 있다.

- Filer 인터페이스
  - 소스 코드, 클래스 코드 및 리소스를 생성할 수 있는 인터페이스

- 유틸리티
  - Javapoet: 소스 코드 생성 유틸리티
    - 간단한 API를 제공하여 Class, Method 등을 생성할 수 있다.
    ```
    <dependency>
		    <groupId>com.squareup</groupId>
		    <artifactId>javapoet</artifactId>
		    <version>1.12.1</version>
		</dependency>
    ```
  
#### 정리

- 애노테이션 프로세서 사용 예
  - 롬복
  - AutoService: java.util.ServiceLoader용 파일 생성 유틸리티
  - @Override
    - https://stackoverflow.com/questions/18189980/how-do-annotations-like-overridework-internally-in-java/18202623
  - Dagger 2: 컴파일 타임 DI 제공
  - 안드로이드 라이브러리
    - ButterKinfe: @BindView (뷰 아이디와 애노테이션 붙인 필드 바인딩)
    - DeepLinkDispatch: 특정 URI 링크를 Activity로 연결할 때 사용

- 애노테이션 프로세서 장점
  - 런타임 비용이 제로

- 애노테이션 프로세서 단점
  - 기존 클래스 코드를 변경할 때는 약간의 hack이 필요하다.
