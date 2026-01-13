---
title: RUST + JNI를 활용한 Password Encoder 만들기
date: 2026-01-13 09:00:00 +09:00
published: true
toc_sticky: true
mermaid: true
categories: [사이드 프로젝트, 추가개발]
tags: [Java, Rust, JNI, Argon2, Spring Boot]
---

## 📋 목차

- [개요](#개요)
- [문제 상황](#문제-상황)
- [해결 과정](#해결-과정)
- [성능 비교](#성능-비교)
- [프로젝트 개선 계획](#프로젝트-개선-계획)
- [참고 자료](#참고-자료)

---

## 개요

Spring Boot 프로젝트에서 비밀번호 해싱을 구현하면서 성능 문제에 직면했습니다.<br>
Spring Security의 `Argon2PasswordEncoder`는 순수 Java 구현으로, 비밀번호 해싱에 **약 450ms**가 소요되었습니다. 추가적으로 **Spring 5.8버전 이상에서 사용 가능**하다는 문제가 있었습니다.

이 문제를 해결하기 위해 Rust로 Argon2 해싱을 구현하고 JNI(Java Native Interface)를 통해 Spring Boot와 연동하는 라이브러리를 개발했습니다. 그 결과 **2.6배의 성능 향상**(450ms → 170ms)을 달성했고, 이 과정을 공유하고자 합니다.

<br>

## 문제 상황

### 기존 방식의 한계

**Spring Security Crypto의 Argon2PasswordEncoder:**
```java
@Configuration
public class SecurityConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
    }
}
```

**성능 측정 결과:**
```
해싱 시간: 평균 450ms
처리량: 약 2.22 해시/초
```

회원가입 10명을 동시에 처리할 경우 **4.5초**가 소요

### 대안 탐색

1. **argon2-jvm**: 더 빠른 Java 구현체
   - 문제: Spring Boot 3.x와 호환성 문제
   - 결과: 프로젝트에서 제외

2. **bcrypt 사용**: 더 빠른 알고리즘으로 변경
   - 문제: Argon2는 OWASP에서 권장하는 최신 표준 **(bcrypt는 1999년 출시)**
   - 결과: 보안성 유지를 위해 Argon2 고수

3. **Rust + JNI**: 네이티브 구현 활용
   - 장점: C/Rust 구현체의 높은 성능
   - 장점: 메모리 안전성 보장
   - 선택!

<br>

## 해결 과정

### 1. Rust 라이브러리 구현

**프로젝트 구조:**
```
argon2-rust-jni/
├── rust/               # Rust 구현
│   ├── src/
│   │   └── lib.rs
│   └── Cargo.toml
└── java/src/main               # Java 래퍼
    └── java/
    │   └── io/github/kitstu17/argon2/
    │       ├── Argon2Encoder.java
    │       └── NativeLibraryLoader.java
    └── resources/natives/      # 네이티브 라이브러리
```

**Rust 구현 일부(lib.rs):**

```rust
...
// Argon2 생성
let argon2 = Argon2::new(
    Algorithm::Argon2id,
    Version::V0x13,
    params
);

// Salt 생성 및 해싱
let salt = SaltString::generate(&mut OsRng);
let password_hash = argon2
    .hash_password(password.as_bytes(), &salt)
    .unwrap()
    .to_string();
...
```

### 2. Rust 코드 로더 구현

**NativeLibraryLoader.java 일부:**

멀티 플랫폼 지원을 위한 동적 라이브러리 로더:
```java
public static synchronized void load() {
...
    // detectPlatform()로 실행 환경의 OS와 Architecture를 가져온다.
    String platform = detectPlatform();
    String libraryFileName = getLibraryFileName();
    String resourcePath = "/natives/" + platform + "/" + libraryFileName;

    // JAR 내부 리소스 확인
    InputStream libraryStream = NativeLibraryLoader.class.getResourceAsStream(resourcePath);

    // 임시 파일로 추출
    Path tempFile = extractLibrary(libraryStream,libraryFileName);

    // 네이티브 라이브러리 로드
    System.load(tempFile.toAbsolutePath().toString());
    loaded = true;
}
...
```

### 3. Maven Central에 배포하기

**build.gradle 일부**
```gradle
plugins {
    id 'java-library'
    id 'signing'   //GPG 서명을 위한 플러그인
    id 'com.vanniktech.maven.publish' version '0.30.0'  // Maven Central 배포를 위한 서드파티 플러그인
}

...

mavenPublishing {
    publishToMavenCentral(com.vanniktech.maven.publish.SonatypeHost.CENTRAL_PORTAL, true)
    signAllPublications()

    pom {
        name = 'Argon2 Rust JNI'
        url = 'https://github.com/kitstu17/argon2-rust-jni'
        inceptionYear = '2025'

        // 개발자 정보, 라이센스, scm도 같이 넣어야 한다.(해당 부분 생략)
        ...
    }
}
```

### 4. Spring Boot 프로젝트와 통합

**build.gradle 일부**

해당 라이브러리를 로드할 프로젝트의 build.gradle:
```gradle
dependencies {
    ...
    implementation 'io.github.kitstu17:argon2-rust-jni:0.1.2'
    ...
}
```

**SecurityConfig 일부**

기존 Spring Security 설정 클래스에 추가하였다.
```java
...
@Bean
public PasswordEncoder passwordEncoder() {
    return new PasswordEncoder() {
        private final Argon2Encoder encoder = new Argon2Encoder(16384, 2, 1);

        @Override
        public String encode(CharSequence rawPassword) {
            return encoder.encode(rawPassword);
        }

        @Override
        public boolean matches(CharSequence rawPassword, String encodedPassword) {
            return encoder.matches(rawPassword, encodedPassword);
        }
    };
}
...
```

<br>

## 성능 비교

### 벤치마크 결과

**테스트 환경:**
- CPU: Intel i5-1240P (12코어)
- RAM: 16GB
- OS: Windows 11
- 테스트: 100회 해싱

| 구현체 | 평균 시간 | 처리량 | 개선율 |
|--------|-----------|--------|--------|
| Spring Security Crypto | 450ms | 2.22 해시/초 | - |
| **argon2-rust-jni** | **170ms** | **5.88 해시/초** | **2.6배** |

<br>

## 프로젝트 개선 계획

### 단기 (1-2개월)

1. **macOS Intel 지원**: x86_64-apple-darwin 추가

### 장기 (3-6개월)

1. **다른 해싱 알고리즘 추가**: bcrypt, scrypt

## 결론

Rust와 JNI를 활용하여 **2.6배 빠른** 비밀번호 해싱 라이브러리를 개발했습니다. 이 과정에서:

- ✅ **성능**: 450ms → 170ms (2.6배 향상)
- ✅ **보안**: OWASP 권장사항 준수
- ✅ **안정성**: Rust의 메모리 안전성
- ✅ **사용성**: Spring Boot와 완벽한 통합
- ✅ **배포**: Maven Central 공개

를 경험해 볼 수 있었습니다.

## 참고 자료

- **GitHub Repository**: [https://github.com/kitstu17/argon2-rust-jni](https://github.com/kitstu17/argon2-rust-jni)
- **Maven Central**: [https://central.sonatype.com/artifact/io.github.kitstu17/argon2-rust-jni](https://central.sonatype.com/artifact/io.github.kitstu17/argon2-rust-jni)
- **OWASP Password Storage**: [https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- **Argon2 RFC**: [https://datatracker.ietf.org/doc/html/rfc9106](https://datatracker.ietf.org/doc/html/rfc9106)

---