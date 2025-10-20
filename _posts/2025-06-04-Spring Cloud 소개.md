---
title: MSA 프로젝트 진행을 위한 Spring Cloud 정리
date: 2025-06-04 19:30:00 +09:00
published: true
toc_sticky: true
categories: [MSA]
tags: [Spring WebFlux, Reactive]
---

## Spring Cloud란?

Spring Cloud는 분산 시스템 개발을 위한 포괄적인 프레임워크로, 마이크로서비스 아키텍처(MSA) 구축에 필요한 다양한 패턴과 도구를 제공
</br></br>
Spring Boot 기반으로 구축되어 있어 빠르고 쉽게 분산 시스템을 개발 가능


## 주요 특징

### 1. 서비스 디스커버리 (Service Discovery)
- 마이크로서비스 간 동적 서비스 탐색
- 서비스 인스턴스의 자동 등록 및 해제
- 클라이언트 측 로드 밸런싱

### 2. 라우팅 (Routing)
- API Gateway 패턴 구현
- 동적 라우팅 및 필터링
- 요청/응답 변환

### 3. 서비스 간 호출 (Service-to-Service Calls)
- 선언적 REST 클라이언트
- 로드 밸런싱된 서비스 호출
- 서킷 브레이커 패턴

### 4. 부하 분산 (Load Balancing)
- 클라이언트 측 로드 밸런싱
- 다양한 로드 밸런싱 알고리즘 지원
- 헬스 체크 기반 인스턴스 선택


## Spring Cloud 주요 프로젝트

### Spring Cloud Config
- 분산 시스템의 외부 설정을 중앙에서 관리
- Git, SVN, 파일시스템 등 다양한 저장소 지원
- 설정 변경 시 애플리케이션 재시작 없이 반영 가능

### Spring Cloud Gateway
- **WebFlux 기반의 리액티브 API Gateway**
- 라우팅, 필터링, 속도 제한, 서킷 브레이커 등 제공
- Netty 런타임 위에서 동작하는 비동기/논블로킹 아키텍처

### Spring Cloud Netflix

Netflix는 자사의 대규모 분산 시스템 운영 경험을 바탕으로 다양한 오픈소스 프로젝트를 공개했고, Spring Cloud는 이를 Spring 생태계에 통합됨.

### Spring Cloud Consul
- HashiCorp Consul 기반 서비스 디스커버리
- 분산 설정 관리
- 헬스 체크 및 키-값 저장소

## 프로젝트 진행 시 유의할 점

### Netflix와 Consul의 현황

**Netflix**
- Spring Cloud Greenwich Release Train (2018년 12월)부터 일부 컴포넌트가 Maintenance Mode로 전환
- Netflix 내부에서도 2016년경부터 일부 컴포넌트의 활발한 개발이 중단됨
- Spring Cloud Ilford Release Train에서 Ribbon, Hystrix, Zuul 완전 제거 예정

**Consul**
- ✅ **완벽하게 지원됨**
- Spring Cloud Consul은 리액티브 환경을 완전히 지원
- WebFlux 기반 Spring Cloud Gateway와 원활하게 통합

<span style="color: orange">**프로젝트 진행 시 Spring Cloud Netflix의 경우, Eureka외의 컴포넌트가 Maintenance mode로 전환된 것을 알고 진행할 필요가 있을 듯 하다.**</span>

## Spring Cloud 2025 현황

### 최신 버전
- **Spring Cloud 2025.0.0 (Northfields)** 출시
- Spring Boot 3.5.0과 호환
- Spring Framework 6.x 기반

### 주요 변화
1. **Gateway 모듈 재구성**
   - `spring-cloud-gateway-server-webflux` (리액티브)
   - `spring-cloud-gateway-server-webmvc` (서블릿)

2. **LoadBalancer 개선**
   - Spring Cloud LoadBalancer가 기본 로드 밸런서로 확립
   - Ribbon 완전 제거 권장

3. **Resilience4j 통합 강화**
   - Hystrix 대체로 Resilience4j 사용 권장
   - Circuit Breaker, Rate Limiter, Retry, Bulkhead 등 지원

## 참고 자료

- [Spring Cloud Official Documentation](https://spring.io/projects/spring-cloud)
- [Spring Cloud 2025.0.0 Release Notes](https://spring.io/blog/2025/05/29/spring-cloud-2025-0-0-is-abvailable/)
- [Spring Cloud Gateway Reference](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- [HashiCorp Consul Documentation](https://www.consul.io/docs)
- [Resilience4j Documentation](https://resilience4j.readme.io/)