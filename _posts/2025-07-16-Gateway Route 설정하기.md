---
title: Spring Cloud Gateway 라우팅 설정하기
date: 2025-07-16 19:30:00 +09:00
published: true
toc_sticky: true
categories: [MSA]
tags: [Spring WebFlux, Reactive]
---

## 목차
1. [라우팅 개요](#1-라우팅-개요)
2. [RouteLocator와 RouteLocatorBuilder](#2-routelocator와-routelocatorbuilder)
3. [Route 구성 요소](#3-route-구성-요소)
4. [RouteConfig 구현](#4-routeconfig-구현)
5. [Predicate 설정](#5-predicate-설정)
6. [Filter 설정](#6-filter-설정)
7. [참고 자료](#7-참고-자료)

---

## 1. 라우팅 개요

### Spring Cloud Gateway 라우팅이란?

Spring Cloud Gateway는 클라이언트의 요청을 받아 적절한 마이크로서비스로 전달하는 역할을 수행.
</br> 이 과정을 **라우팅(Routing)** 이라고 하며, 다음과 같은 흐름으로 동작한다.:

```
클라이언트 요청
    ↓
Gateway (Route 매칭)
    ↓
Predicate 평가 (경로, 헤더, 메서드 등)
    ↓
Filter 체인 실행 (요청 변환, 인증, 로깅 등)
    ↓
Target Service (마이크로서비스)
    ↓
Filter 체인 실행 (응답 변환, 로깅 등)
    ↓
클라이언트 응답
```

### 라우팅 설정 방식

Spring Cloud Gateway는 두 가지 방식으로 라우팅을 설정 가능:

1. **Java Config (권장)**: `RouteLocator` Bean을 통한 프로그래밍 방식
2. **Properties/YAML**: `application.yml`에 선언적으로 설정

해당 프로젝트는 Java Config 방식으로 진행되었다.

---

## 2. RouteLocator와 RouteLocatorBuilder

### RouteLocator

`RouteLocator`는 Gateway의 모든 라우팅 규칙을 담는 인터페이스

```java
public interface RouteLocator {
    Flux<Route> getRoutes();  // 모든 Route를 반환
}
```

### RouteLocatorBuilder

`RouteLocatorBuilder`는 라우팅 규칙을 쉽게 작성할 수 있도록 도와주는 빌더 클래스

```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("route-id", r -> r
            .path("/api/**")          // Predicate: 경로 매칭
            .filters(f -> f.addRequestHeader("X-Custom", "Value"))  // Filter
            .uri("http://example.com"))  // Target URI
        .build();
}
```

## 3. Route 구성 요소

하나의 Route는 다음 3가지 요소로 구성됩니다:

### 1. Route ID
- Route를 식별하는 고유 ID
- 로그, 모니터링, 디버깅에 사용

```java
.route("admin-service", r -> ...)  // "admin-service"가 Route ID
```

### 2. Predicate (조건)
- 요청이 이 Route에 매칭되는지 판단하는 조건
- 경로, 헤더, 쿼리 파라미터, 메서드 등으로 매칭

```java
r.path("/admin/**")           // 경로 기반
r.header("X-Request-Id")      // 헤더 기반
r.method("GET", "POST")       // 메서드 기반
r.query("userId")             // 쿼리 파라미터 기반
```

### 3. URI (목적지)
- 요청을 전달할 타겟 서비스의 주소
- HTTP(S) URL 또는 Service Discovery 기반 로드 밸런싱

```java
.uri("http://localhost:8081")           // 직접 URL
.uri("lb://admin-service")              // 로드 밸런싱 (Consul/Eureka)
```

### 4. Filters (선택사항)
- 요청/응답을 변환하거나 추가 처리를 수행
- 인증, 로깅, 헤더 추가, 경로 재작성 등

```java
.filters(f -> f
    .addRequestHeader("X-Custom", "Value")
    .rewritePath("/admin/(?<path>.*)", "/${path}")
    .circuitBreaker(c -> c.setName("adminCB")))
```

## 4. RouteConfig 구현

### 전체 RouteConfig 예제 (실습 중인 프로젝트)

- 해당 프로젝트는 현재 Gateway Service만 존재
- 추후 운영자, 사용자 분리를 위한 admin, user 서비스 개발 진행 예정

```java
...
@Configuration
@RequiredArgsConstructor
public class RouteConfig {

    private final AuthGlobalFilter authFilter;
    private static final String REWRITE_REPLACE_STRING = "/${path}";
    private static final String CALL_ORIGIN_NAME = "call-origin";
    private static final String CALL_ORIGIN_VALUE = "gateway-service";

    // Admin Service 설정
    @Value("${cloud.admin.service.uri}")
    private String adminUri;
    @Value("${cloud.admin.service.path}")
    private String[] adminServicePath;
    @Value("${cloud.admin.service.rewrite.pattern}")
    private String adminRewritePtn;

    // User Service 설정
    @Value("${cloud.user.service.uri}")
    private String userUri;
    @Value("${cloud.user.service.path}")
    private String[] userServicePath;
    @Value("${cloud.user.service.rewrite.pattern}")
    private String userRewritePtn;

    @Bean
    public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("admin-service", r -> r.path(adminServicePath)
                        .filters(f -> f
                                .addRequestHeader(CALL_ORIGIN_NAME, CALL_ORIGIN_VALUE)
                                .rewritePath(adminRewritePtn, REWRITE_REPLACE_STRING)
                                .circuitBreaker(c -> c
                                        .setName("adminServiceCircuitBreaker")
                                        .setFallbackUri("forward:/fallback/admin-service")))
                        .uri(adminUri))

                .route("user-service", r -> r.path(userServicePath)
                        .filters(f -> f
                                .addRequestHeader(CALL_ORIGIN_NAME, CALL_ORIGIN_VALUE)
                                .rewritePath(userRewritePtn, REWRITE_REPLACE_STRING)
                                .circuitBreaker(c -> c
                                        .setName("userServiceCircuitBreaker")
                                        .setFallbackUri("forward:/fallback/user-service")))
                        .uri(userUri))

                .build();
    }
}
```

### application.properties 설정

```properties
# Admin Service 라우팅 설정
cloud.admin.service.uri=lb://admin-service
cloud.admin.service.path=/admin/**
cloud.admin.service.rewrite.pattern=/admin/(?<path>.*)

# User Service 라우팅 설정
cloud.user.service.uri=lb://user-service
cloud.user.service.path=/user/**
cloud.user.service.rewrite.pattern=/user/(?<path>.*)
```

---

## 5. Predicate 설정

현재 프로젝트에서 적용한 방법은 경로 기반 설정 방식이다.

```java
// 단일 경로
.route("api", r -> r.path("/api/users")
    .uri("lb://user-service"))

// 와일드카드
.route("api", r -> r.path("/api/**")
    .uri("lb://user-service"))

// 여러 경로
.route("api", r -> r.path("/api/**", "/v1/**")
    .uri("lb://user-service"))

// @Value로 외부화
@Value("${cloud.admin.service.path}")
private String[] adminServicePath;  // ["/admin/**"]

.route("admin", r -> r.path(adminServicePath)
    .uri(adminUri))
```

## 6. Filter 설정

### 주요 Filter Factory

#### 1. AddRequestHeader (요청 헤더 추가)

```java
.filters(f -> f
    .addRequestHeader("X-Request-Source", "gateway")
    .addRequestHeader("X-Gateway-Time", LocalDateTime.now().toString()))
```

**본 프로젝트 활용**:
```java
.filters(f -> f
    .addRequestHeader("call-origin", "gateway-service"))
```
- **목적**: 백엔드 서비스에서 Gateway를 통해 들어온 요청임을 식별
- **활용**: 로깅, 모니터링, 내부 서비스 간 통신 제어

#### 2. RewritePath (경로 재작성)

```java
// 기본 형식
.filters(f -> f
    .rewritePath("/api/(?<segment>.*)", "/${segment}"))

// 예시
// 클라이언트 요청: GET /api/users/123
// 실제 전달: GET /users/123
```

**본 프로젝트 활용**:
```java
// application.properties
cloud.admin.service.rewrite.pattern=/admin/(?<path>.*)

// RouteConfig.java
.filters(f -> f
    .rewritePath(adminRewritePtn, "/${path}"))

// 예시:
// 클라이언트: GET /admin/users/123
// 실제 전달: GET /users/123
```

## 7. 참고 자료
- [Spring Cloud Gateway Reference](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- [Spring Cloud Gateway Route Predicate Factories](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories)
- [Spring Cloud Gateway Filter Factories](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories)