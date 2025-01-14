---
layout: post
title: "[JAVA] Container 환경에 최적화된 GraalVM과 Quarkus 사용 방법 정리"
tags: [BackEnd JAVA]
---

## Intro

안녕하세요 **Noah**입니다

오늘은 GraalVM에 대해 알아보도록 하겠습니다.
GraalVM은 Java, Javascript, Python, Ruby, 등을 지원하는 Virtual Machine입니다.<br/>

[Graalvm 홈페이지](https://www.graalvm.org/java/)<br/>
<img src="../../../assets/img/2022-07-29-GraalVM/GraalVM.png"  width="700"/>

기존 Java에는 JVM이 존재했습니다. Javac를 통해 컴파일된 Class파일들은 JVM 내 Class Loader를 통해 Class파일들을 JVM 내부에 할당합니다.<br/>
이후 필요한 class 파일들을 Execution Engine을 통해 해석하고 Runtime Area에 배치함으로써 Application이 실행되게 됩니다.<br/>
Application이 작동하는 동안 JVM은 GC를 이용해 메모리 관리를 하기도 하고 Thread 싱크를 맞춰주기도 합니다.

GraalVM은 Java로 구현된 HotSpot/OpenJDK 기반의 Java VM 및 JDK를 지원하는 오픈소스입니다. JDK 8, 11 버전과 호환이 가능하기 때문에 따로 소스코드를 변환할 필요가 없습니다.
또한 Spring, Tomcat 등 기존 Java 서버 개발자분들과 친숙한 Framework들과 호환성도 좋아 한번 사용해 보시는 걸 추천드립니다.

이제 GraalVM을 왜 사용해야 하는지 그리고 GraalVM의 등장으로 어떤 것들이 새롭게 등장했는지 살펴보도록 하겠습니다.
<br/><br/><br/><br/>

## GraalVM은 왜 등장했을까?
기존 레거시 환경에서는 큰 문제가 없었던 OpenJDK였지만 Docker/Kubernatis 등이 등장하며 컨테이너 작업이 많아지는 시점부터 로딩 속도가 느리다는 단점이 계속 지적되어 왔습니다.
이런 이슈를 해결하고자 Oracle에서는 New JIT Compiler인 Graal Compiler를 선보였습니다.

이 Graal Compiler의 특징은 다음과 같습니다.
1. Garbage Collection 알고리즘 개선
2. Runtime Memory 구조 개선
3. Java Library 효율성 향상
4. CPU 최적화
5. Just In Time(JIT) 컴파일러 도입

위 내용들을 통해 Java의 성능이 대폭 상승하였고 아래와 같은 부가 효과를 가져오게 되었습니다.

1. 빠른 스타트 시간
2. 단위 시간당 최대 처리량 증가
3. 동일 작업 간 낮은 메모리 사용량
4. 패키지 파일 용량 감소
5. 지연시간 최소화

위 내용 모두가 다 Container 환경에서 작업을 할 때 필요한 요소들이라 사람들은 
GraalVM은 컨테이너 환경에 대응하기 위해 Oracle이 새롭게 VM을 개발한 것이라고 이야기가 나오는 것 같습니다.

> 더 자세한 내용은 아래 문서를 참고해보세요 ^^
> [문서](http://www.regist-event.com/oracle/2019/mcd_down/pdf/Day1_Track4/Day1_T4_S1.%20Cloud%20Native%20Java%20GraalVM%20-%20%EA%B9%80%ED%83%9C%EC%99%84%20%EB%B6%80%EC%9E%A5.pdf)

## GraalVM의 장점
GraalVM의 장점은 크게 3가지입니다.
1. 인프라 비용 절감
   1. 기존 JVM보다 더 높은 애플리케이션 성능과 더 낮은 CPU 및 메모리 사용량을 제공하기 때문에 인프라에 대한 비용을 절감할 수 있게 됩니다.
2. 코드 변경 없이 더 나은 성능 제공
   1. OpenJDK 8, 11 버전에 맞게 설치를 제공하며 이들과 완벽하게 호응되어 코드 변경없이 서비스를 마이그레이션 가능합니다.
3. 여러 고성능 Framework 대응
   - Micronaut, Helidon, <strong style="color: #bb4177;">'Quarkus'</strong>, or Spring Boot for microservices, PicoCLI 등을 지원합니다.
4. Java 생태계를 확장 가능
   - GraalVM을 사용하면 JVM 위에서 다른 언어들을 사용 가능하며 이를 통해 다양한 서비스를 생산해 낼 수 있습니다.
      - Scala, Kotlin, Clojure, JavaScript/Node.js, Ruby, Python, R 등의 언어를 JVM 위에서 가용하도록 GraalVM이 지원합니다.
      - C, C++과 같은 컴파일 언어들도 LLVM bitcode 또는 WebAssembly(Wasm)로 컴파일하여 사용할 수 있도록 지원합니다.
      - 이를 통해 Python 기계 학습 기능을 통합하는 Java 마이크로서비스 제작 <br/>또는 통계 데이터 분석에 R을 사용하고 JavaScript를 이용해 프로그래밍한 서비스 등을 지원할 수 있게 됩니다. 


## GraalVM과 Quarkus를 활용해 간단히 서버 띄우는 방법
GraalVM을 Container에서 활용할 수 있도록 이번엔 Docker를 활용해서 GraalVM을 사용하겠습니다.

- 먼저 개발을 위해 GraalVM, Quarkus 설치<br/>
  Quarkus 공식 문서 참고<br/>
  > [Quarkus 공식 문서](https://quarkus.io/guides/building-native-image)
  
  <br/>
  Quarkus 공식 문서와 연관된 GraalVM 설치 URL(OS맞게 설치)<br/>
  > [Quarkus 공식 문서](https://github.com/graalvm/graalvm-ce-builds/releases)

<br/>

- GraalVM 설치 후 환경변수 설정<br/>
  GRAALVM_HOME 환경변수와 JAVA_HOME의 환경변수 값을 GraalVM설치 위치로 지정해주시면 됩니다.<br/>
  > <img src="../../../assets/img/BackEnd/Java/2022-07-29-GraalVM/env.jpg"  width="700"/>

<br/>

- Intellij를 활용해서 Quarkus 프로젝트 생성<br/>
  여기서는 Gradle을 활용했습니다.(Maven 사용하셔서 만드셔도 괜찮습니다.)<br/>
  SDK는 아까 다운로드받은 GraalVM의 디렉토리 Path를 잡아주시면 됩니다.<br/>
  > <img src="../../../assets/img/BackEnd/Java/2022-07-29-GraalVM/quarkus_new_project.png"  width="700"/>

<br/>

- Quarkus 설치<br/>
  각 OS맞게 설치해 주세요
  > [Quarkus 설치](https://quarkus.io/get-started/)
  저는 Window라 Powershell을 활용하여 설치하였습니다.(혹시 모르니 관리자 권한으로 실행해서 설치해주세요)

<br/>
  
- quarkus로 서버 실행<br/>
  설치 후 Powershell을 한번 닫아주시고 다시 열어 위에서 생성 프로젝트의 root 위치로 이동합니다.<br/>
  해당 위치에서 'quarkus create' 실행<br/>
  완료되면 'quarkus dev' 실행<br/>
  이렇게 실행하면 http://localhost:8080 으로 서버가 오픈됩니다.<br/>
  참 쉽죠?<br/>
  > <img src="../../../assets/img/BackEnd/Java/2022-07-29-GraalVM/server_open.png"  width="700"/>

<br/>

- Quarkus를 사용하는 Docker 기본 이미지 생성<br/>
  > 참고한 공식 사이트 <br/>
  > https://quarkus.io/guides/building-native-image
  
  <br/><br/>
  
  1. 생성 이유<br/>
     - Dockerfile Path<br/>
       > <img src="../../../assets/img/BackEnd/Java/2022-07-29-GraalVM/dockerfile_path.png"  width="700"/>
     <br/>
      
     - quarkus-micro-image 사용이유<br/>
       quarkus-micro-image는 최소한의 서버 실행 요소로 빠르게 서버를 실행하기 위해 사용됩니다. REST API를 Container환경에서 사용하기 위한 최적의 환경을 제공해 줍니다.
  
  <br/><br/>
  
  2. Gradle 기반
  ```dockerfile
  ## Stage 1 : build with maven builder image with native capabilities
  FROM quay.io/quarkus/ubi-quarkus-native-image:22.1-java11 AS build
  USER root
  RUN microdnf install findutils
  COPY --chown=quarkus:quarkus ../../../gradlew /code/gradlew
  COPY --chown=quarkus:quarkus ../../../gradle /code/gradle
  COPY --chown=quarkus:quarkus ../../../build.gradle /code/
  COPY --chown=quarkus:quarkus ../../../settings.gradle /code/
  COPY --chown=quarkus:quarkus ../../../gradle.properties /code/
  USER quarkus
  WORKDIR /code
  COPY ../../../src /code/src
  RUN ./gradlew build -Dquarkus.package.type=native
  
  ## Stage 2 : create the docker final image
  FROM quay.io/quarkus/quarkus-micro-image:1.0
  WORKDIR /work/
  COPY --from=build /code/build/*-runner /work/application
  RUN chmod 775 /work
  EXPOSE 8080
  CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
  ```
  <br/><br/>
  
  3. Maven 기반
  ```dockerfile
  ## Stage 1 : build with maven builder image with native capabilities
  FROM quay.io/quarkus/ubi-quarkus-native-image:22.1-java11 AS build
  COPY --chown=quarkus:quarkus ../../../mvnw /code/mvnw
  COPY --chown=quarkus:quarkus ../../../.mvn /code/.mvn
  COPY --chown=quarkus:quarkus ../../../pom.xml /code/
  USER quarkus
  WORKDIR /code
  RUN ./mvnw -B org.apache.maven.plugins:maven-dependency-plugin:3.1.2:go-offline
  COPY ../../../src /code/src
  RUN ./mvnw package -Pnative
  
  ## Stage 2 : create the docker final image
  FROM quay.io/quarkus/quarkus-micro-image:1.0
  WORKDIR /work/
  COPY --from=build /code/target/*-runner /work/application
  
  # set up permissions for user `1001`
  RUN chmod 775 /work /work/application \
  && chown -R 1001 /work \
  && chmod -R "g+rwX" /work \
  && chown -R 1001:root /work
  
  EXPOSE 8080
  USER 1001
  
  CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
  ```

<br/><br/><br/><br/>

## 글을 마치며 
처음 다뤄보는 GraalVM, Quarkus라 사용 방식을 익히는데 3일 정도의 시간이 소요되었는데요

아직 레퍼런스가 많지 않아 참고하실만한 내용들이 적기는 합니다만 서버 자체가 로딩되는 속도는 진짜 상당히 빠릅니다. 

사용해 보면서 Quarkus에 대해 놀라게 되었는데 Spring과 다르게 빠른 서버 오픈 속도, 손쉽게 간결한 REST API 제작 가능 등 

Micro 서비스를 만들기에 최적화된 기능들이 많은 Framework라는 생각이 들었습니다.

여러분도 한번 사용해 보시고 내용 공유해 주시면 감사하겠습니다.

다음번에는 FastAPI와 비교해 보고 어떤 것이 더 Container 환경에서 개발하기 적합한지 그리고 더 손쉽게 접근할 수 있는 Framework는 무엇인지 내용을 공유 드기로 하겠습니다.

긴 글 읽어주셔서 감사합니다.