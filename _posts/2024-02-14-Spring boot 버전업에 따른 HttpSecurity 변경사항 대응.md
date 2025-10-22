---
title: Spring boot 버전업에 따른 HttpSecurity 변경사항 대응
date: 2024-02-14 10:30:00 +09:00
published: true
toc_sticky: true
categories: [개발]
tags: [Spring, Spring Security]
---

# Spring boot 버전 변경에 따른 HttpSecurity 문제

## 문제

### HttpSecurity 설정

- Spring boot 버전이 3.1.2로 변경됨에 따라 SecurityFilterChain을 이전과 같이 사용 불가
- 따라서 아래의 북마크를 참고하여 다음과 같이 변경함

    [Configuration Migrations :: Spring Security](https://docs.spring.io/spring-security/reference/migration-7/configuration.html)

    [Configuration Migrations :: Spring Security](https://docs.spring.io/spring-security/reference/5.8/migration/servlet/config.html)

---

## 문제 해결 과정

### 기본 환경 설정

```java
1. 변경 이전(환경 설정)
@Bean
protected void configure(HttpSecurity http) throws Exception {
	http.cors().and()
			.csrf().disable().and()
			.httpBasic().disable().and()
			.sessionManagement()
				.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
			...
}

2. 변경 이후(환경 설정)
@Bean
protected SecurityFilterChain configure(HttpSecurity http) throws Exception {
	http.csrf(csrf -> csrf.disable());
	http.cors(cors -> cors.configurationSource(corsConfigurationSource()));
	http.httpBasic(basic -> basic.disable());
	http.sessionManagement(session -> {
			session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
	});
	...

	return http.build();
}

@Bean
public CorsConfigurationSource corsConfigurationSource() {
	final long MAX_AGE_SECS = 3600;
	UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
	CorsConfiguration config = new CorsConfiguration();
	config.setAllowCredentials(true);
	config.addAllowedOriginPattern("*");
	config.addAllowedHeader("*");
	config.addAllowedMethod("GET");
	...
	config.setMaxAge(MAX_AGE_SECS);
	source.registerCorsConfiguration("/**", config);
	return source;
}
```

**주요 변경사항:**
- 버전이 변경됨에 따라 람다식으로 표현하도록 변경되었다.
- 이전과 같이 and() 메소드를 사용하여 표현할 수 없다.
- cors() 메소드의 경우, WebMvcConfigurer를 상속하여 CorsConfig를 설정하는 방식의 사용이 불가능하다.
- CorsConfigurationSource를 반환하는 @Bean을 제작하여 등록하는 방식을 사용한다.

---

### 시큐리티 설정

#### 접근 권한 설정 1

```java
1. 변경 이전(시큐리티 설정 - 접근 권한1)
@Bean
protected void configure(HttpSecurity http) throws Exception {
	http.authorizeRequests()
			.antMatchers("/user/**").authenticated()
			.antMatchers("/admin/**").access("hasRole('ROLE_ADMIN')")
			.antMatchers("/admin/**")
			.access("hasRole('ROLE_ADMIN') and hasRole('ROLE_USER')")
			.anyRequest().permitAll()
}

2. 변경 이후(시큐리티 설정 - 접근 권한1)
@Bean
protected SecurityFilterChain configure(HttpSecurity http) throws Exception {
	http.authorizeHttpRequests(auth -> {
			auth.requestMatchers("/user/**").authenticated()
				.requestMatchers("/admin/**")
				.access(new WebExpressionAuthorizationManager("hasRole('ROLE_ADMIN')"))
				.requestMatchers("/manager/**")
				.access(new WebExpressionAuthorizationManager
						("hasRole('ROLE_MANAGER') or hasRole('ROLE_ADMIN')"))
				.anyRequest().permitAll();
	});
	...
	return http.build();
}
```

**주요 변경사항:**
- 기존의 authorizeRequests() 메소드가 authorizeHttpRequests()로 변경됨
- antMatchers()로 연결하던 방식이 람다식과 requestMatchers()를 사용하는 방식으로 변경됨
- access()메소드의 사용 방식이 WebExpressionAuthorizationManager 객체를 같이 사용하는 방식으로 변경됨

#### 접근 권한 설정 2

```java
1. 변경 이전(시큐리티 설정 - 접근 권한2)
@Bean
protected void configure(HttpSecurity http) throws Exception {
	http.authorizeRequests()
			.antMatchers("/","/auth/**","/h2-console/**").permitAll()
			.anyRequest().authenticated()
}

2. 변경 이후(시큐리티 설정 - 접근 권한2)
@Bean
protected SecurityFilterChain configure(HttpSecurity http) throws Exception {
	http.authorizeHttpRequests(auth -> {
		auth.requestMatchers(new AntPathRequestMatcher("/"),
					new AntPathRequestMatcher("/auth/**"),
					new AntPathRequestMatcher("/h2-console/**")).permitAll();
	});
	...
	return http.build();
}
```

**주요 변경사항:**
- 로그인 없이 접근 가능 API 설정 방법 변경
- 기존의 antMatchers() 메소드 사용 불가
- 람다식, requestMatchers()메소드, AntPathRequestMatcher 객체를 사용하는 방식으로 변경됨

---

### 로그인 처리

#### Form Login

```java
1. 변경 이전(로그인 처리1)
@Bean
protected void configure(HttpSecurity http) throws Exception {
	http.formLogin()
			.loginPage("/loginForm")
			.loginProcessingUrl("/login")
			.defaultSuccessUrl("/")
	...
}

2. 변경 이후(로그인 처리1)
@Bean
protected SecurityFilterChain configure(HttpSecurity http) throws Exception {
	http.formLogin(login -> login.loginPage("/loginForm")
															.loginProcessingUrl("/login")
															.defaultSuccessUrl("/"));
	...
	return http.build();
}
```

**주요 변경사항:**
- 이전과 달리 formLogin() 메소드 내부에서 람다식으로 표현

#### OAuth2 Login

```java
1. 변경 이전(로그인 처리2)
@Bean
protected void configure(HttpSecurity http) throws Exception {
	http.oauth2Login()
			.loginPage("/loginForm")
			.userInfoEndpoint()
			.userService(principalOauth2UserService);
	...
}

2. 변경 이후(로그인 처리2)
@Bean
protected SecurityFilterChain configure(HttpSecurity http) throws Exception {
	http.oauth2Login(oauth -> {
		oauth.loginPage("/loginForm")
				.userInfoEndPoint(info -> {
			info.userService(principalOauth2UserService);
		});
	});
	...
	return http.build();
}
```

**주요 변경사항:**
- oauth2Login() 메소드의 경우 다른 메소드들과 동일하게 람다식을 사용하여 내부에서 표현하도록 변경됨
