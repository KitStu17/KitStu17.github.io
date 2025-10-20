---
title: Spring Cloud Gateway 프로젝트 만들기
date: 2025-07-02 19:30:00 +09:00
published: true
toc_sticky: true
categories: [MSA]
tags: [Spring WebFlux, Reactive]
---

## 프로젝트 생성하기

Spring Cloud Gateway 프로젝트를 생성하기 위해 [Spring Initializr](https://start.spring.io/)에서 다음과 같이 설정하였습니다.:

**프로젝트 메타데이터**
- Project: `Gradle - Groovy`
- Language: `Java`
- Spring Boot: `3.5.6`
- Java: `17`
- Packaging: `Jar`

**프로젝트 정보**
- Group: `com.cloud`
- Artifact: `gateway`
- Name: `gateway`
- Package name: `com.cloud.gateway`

**기본 Dependencies**
- Spring Reactive Web (WebFlux)
- Spring Cloud Gateway
- Spring Cloud Consul Discovery
- Spring Boot Actuator
- Spring Security

---


## build.gralde 수정

프로젝트 생성 후 swagger, jaspyt등 다른 의존성 추가 후 모습
```gradle
// dependencies 만 추가하고 나머지 생략
dependencies {
    // ===================================
    // 핵심 Spring Cloud Gateway 의존성
    // ===================================

    // 1. WebFlux (리액티브 웹 스택)
    implementation 'org.springframework.boot:spring-boot-starter-webflux'

    // 2. Spring Cloud Gateway (WebFlux 기반)
    implementation 'org.springframework.cloud:spring-cloud-starter-gateway-server-webflux'

    // 3. Consul Service Discovery
    implementation 'org.springframework.cloud:spring-cloud-starter-consul-discovery'

    // 4. Spring Security (인증/인가)
    implementation 'org.springframework.boot:spring-boot-starter-security'

    // 5. Actuator (모니터링/헬스체크)
    implementation 'org.springframework.boot:spring-boot-starter-actuator:3.5.6'

    // ===================================
    // Circuit Breaker (Resilience4j)
    // ===================================
    implementation 'org.springframework.cloud:spring-cloud-starter-circuitbreaker-reactor-resilience4j'

    // ===================================
    // 추가 기능 의존성
    // ===================================

    // Database
    implementation 'org.postgresql:postgresql'
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.5'
    implementation 'org.bgee.log4jdbc-log4j2:log4jdbc-log4j2-jdbc4.1:1.16'

    // JWT 토큰
    implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
    implementation 'io.jsonwebtoken:jjwt-impl:0.11.2'
    implementation 'io.jsonwebtoken:jjwt-jackson:0.11.2'

    // Error Handling (Problem Details for HTTP APIs)
    implementation 'org.zalando:problem-spring-webflux:0.29.1'
    implementation 'org.zalando:jackson-datatype-problem:0.27.1'

    // Jasypt (속성 암호화)
    implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.4'

    // Swagger UI (API 문서화)
    implementation 'org.springdoc:springdoc-openapi-starter-webflux-ui:2.8.9'

    // Email
    implementation 'org.springframework.boot:spring-boot-starter-mail:3.4.1'
    implementation 'org.springframework:spring-context-support:6.2.1'
    implementation 'com.sun.mail:jakarta.mail:2.0.1'

    // Lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // Logging
    implementation 'org.codehaus.janino:janino:3.1.12'

    // ===================================
    // 테스트 의존성
    // ===================================
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'io.projectreactor:reactor-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

## Consul과의 연결

### 디렉토리 구조
```
gateway/
└── src/
    └── main/
        └── resources/
            └── config/
                ├── application.properties          # 기본 설정
                └── local/
                    └── app.properties             # 환경별 설정
```

### application.properties (기본 설정)

```properties
# 애플리케이션 이름 (Consul에 등록될 서비스명)
spring.application.name=gateway-service

# 서버 포트
server.port=8080

# 프로파일 설정
spring.profiles.active=local

# 환경별 설정 파일 임포트
spring.config.import=classpath:config/${spring.profiles.active}/app.properties

# MyBatis 설정
mybatis.config-location=mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:sql/postgres/*.xml

# 로그 설정
logging.config=classpath:config/logback-spring.xml
```

### Consul과의 연결을 위한 설정 추가
```properties
# ===================================
# Consul Discovery 설정
# ===================================

# Consul 서버 호스트
spring.cloud.consul.host=localhost

# Consul 서버 포트
spring.cloud.consul.port=9800

# Consul 연결 실패 시 애플리케이션 시작 실패 여부
# true: Consul 연결 실패 시 즉시 실패 (개발 환경 권장)
# false: Consul 연결 실패해도 애플리케이션 시작 (운영 환경 고려)
spring.cloud.consul.fail-fast=true

# Consul 기능 활성화
spring.cloud.consul.enabled=true

# Service Discovery 활성화
spring.cloud.consul.discovery.enabled=true

# Consul Config 비활성화 (필요시 활성화)
spring.cloud.consul.config.enabled=false
```

#### 설정 항목에 대한 설명

| 속성 | 설명 | 권장값 |
|------|------|--------|
| `spring.cloud.consul.host` | Consul 서버 호스트 주소 | `localhost` (개발), 실제 호스트 (운영) |
| `spring.cloud.consul.port` | Consul 서버 포트 | `8500` (기본), `9800` (커스텀) |
| `spring.cloud.consul.fail-fast` | 연결 실패 시 즉시 실패 여부 | `true` (개발), `false` (운영) |
| `spring.cloud.consul.enabled` | Consul 전체 기능 활성화 | `true` |
| `spring.cloud.consul.discovery.enabled` | Service Discovery 활성화 | `true` |
| `spring.cloud.consul.config.enabled` | Consul Config 활성화 | `false` (Config Server 사용 시) |

### Consul 연결 확인

먼저 consul은 따로 다운로드가 필요하다. [Download Consul](https://developer.hashicorp.com/consul/install)
</br></br>
consul 다운로드 후 실행하기(테스트 환경)
```bash
./consul agent -dev -ui -client 127.0.0.1 -http-port 8500
```

## Discovery Client 설정

### GatewayApplication.java

```java
package com.cloud.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient  // Discovery Client 활성화
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```

## 참고 자료

- [Spring Cloud Gateway Reference](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- [Spring Cloud Consul Reference](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/)
- [Consul Documentation](https://www.consul.io/docs)
- [Spring WebFlux Reference](https://docs.spring.io/spring-framework/reference/web/webflux.html)
