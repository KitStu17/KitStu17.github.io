---
title: Spring Boot CDS 적용하기
date: 2024-10-22 19:30:00 +09:00
published: true
toc_sticky: true
categories: [개발, 성능 개선]
tags: [Spring Boot]
---

# CDS란?

- CDS(Class Data Sharing) 는 Java 애클리케이션 시작 시간과 메모리 공간을 줄이는 데 도움이 되는 JVM

- 해당 기능 사용을 위해선 애플리케이션의 특정 클래스 경로에 대해 CDS 아카이브 생성 필요

- Spring Boot 3.3.x 버전 이상부터 해당 기능을 사용할 수 있다.

## Spring Boot 빌드 설정 변경 (Gradle)

- 해당 실습에선 gradle에서 실행하는 것을 전재 합니다.

- 먼저 build.gradle에 다음 설정을 추가하여 실행 가능한 JAR 파일을 만들어야 한다.

    ```groovy
    ...
    tasks.withType(JavaExec) {
        jvmArgs += ['-XX:SharedArchiveFile=app-cds.jsa']
        jvmArgs += ['-XX:+UnlockCommercialFeatures', '-XX:+UseAppCDS']
    }
    ...
    ```

- **tasks.withType(JavaExec)**을 사용하여 JavaExec 유형의 작업들에 일괄적으로 JVM 옵션을 설정하도록 한다.

## CDS 아카이브 생성

- 먼저 JAR 파일을 빌드한다.
    ```bash
        ./gradlew bootJar
    ```

- 이후 JAR 파일을 실행하여 클래스 데이터를 수집한다.
    ```bash
    java -Xshare:dump -XX:SharedArchiveFile=app-cds.jsa -jar build/libs/<자신의 jar파일명>.jar
    ```
- 위의 명령을 사용하여 app-cds.jsa라는 파일을 생성하여 해당 애플리케이션의 클래스 데이터를 저장한다.

## CDS 아카이브 적용하여 실행

- 생성된 CDS 아카이브를 사용하여 애플리케이션을 실행하면 CDS를 활용할 수 있다.
    ```bash
    java -Xshare:on -XX:SharedArchiveFile=app-cds.jsa -jar build/libs/<자신의 jar파일명>.jar
    ```

### build.gradle 적용 예시

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.4'
    id 'io.spring.dependency-management' version '1.1.6'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}

bootJar {
    archiveFileName = 'cds-demo.jar'  // JAR 파일명
    destinationDirectory = file("$buildDir/custom-jar")
}

tasks.register('cds-build', JavaExec) {
    group = 'application' //Task 그룹
    description = 'Builds a custom archive file with AppCDS'

    dependsOn tasks.bootJar

    mainClass.set('com.example.cds_demo.CdsDemoApplication')  // Main 클래스 지정 (프로젝트에 맞게 수정)
    classpath = sourceSets.main.runtimeClasspath
    args=[]

    // CDS 옵션 추가
    jvmArgs += ['-Xshare:dump', '-XX:SharedArchiveFile=build/custom-jar/app-cds.jsa']

    // JAR 파일 위치 설정
    inputs.files("build/custom-jar/cds-demo.jar")  // 생성된 JAR 파일 지정 (파일명 수정 필요)
    outputs.file("build/custom-jar/app-cds.jsa")   // CDS 파일 출력 위치 지정
}
```
