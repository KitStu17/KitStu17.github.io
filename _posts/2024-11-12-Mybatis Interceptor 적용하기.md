---
title: Mybatis Interceptor 적용하기
date: 2024-11-12 19:30:00 +09:00
published: true
toc_sticky: true
categories: [개발, 성능 개선]
tags: [Spring Boot]
---

# Mybatis란?

- Java 애플리케이션에서 데이터베이스와의 상호작용을 쉽게 만들어주는 **ORM(Object Relational Mapping)** 프레임워크

- SQL 쿼리를 직접 작성, 데이터베이스와의 연결을 관리, 매핑을 통해 데이터베이스의 데이터를 Java 객체로 변환하는 작업 수행

- DAO와 DB 사이의 연결에 아래와 같은 과정이 수행됨
    ![Mybatis-1](/assets/img/mybatis/Mybatis-1.png)

## Mybatis의 실행 구조

- SqlSessionFactoryBuilder : 
    1. config.xml에 정의된 Mybatis 연결에 대한 설정을 받아와 SqlSessionFactory를 만든다.

    2. SqlSessionFactoryBuilder는 Bean으로 등록하여 Appication 시작 시 SqlSessionFactory 객체를 생성한다.

- SqlSessionFactory : 
    1. DB와의 연결, DAO에서 정의된 메소드가 실행할 mapper에 대한 정보 등 실제 Application과 DB가 연결될 때 필요한 설정을 가진다.

    2. DAO에서 DB연결 요청이 발생 시 SqlSession 객체를 생성하여 SQL을 실행할 수 있도록 한다.

    3. 따라서 해당 객체에서 **Interceptor**에 대한 정보를 담고있다가 DB연결 요청 발생 시 Interceptor가 추가된 SqlSession을 생성한다.

- SqlSession : 
    1. 실제 DB와 연결되는 객체 

    2. DAO 메서드가 DB연결 요청 시 생성되며 SQL을 실행하고 실행 결과를 매핑하여 Java 객체로 전달한다.

    3. DAO 메서드가 호출될 때마다 새로운 SqlSession이 생성된다.


## Mybatis Interceptor 적용하기

- Mybatis Interceptor를 적용하기 위해 **mybatis-config.xml**에 plugins 설정을 추가할 필요가 있다.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <setting name="mapUnderscoreToCamelCase" value="true" />
            <setting name="callSettersOnNulls" value="true" />
            <setting name="jdbcTypeForNull" value="NULL" />
        </settings>
        <!-- 아래의 plugs 태그가 중요 -->
        <!-- interceptor의 값은 실제 작성한 Interceptor class를 넣어준다. -->
        <plugins>
            <plugin interceptor="com.example.common.interceptor.MyMyBatisInterceptor"/>
        </plugins>
    </configuration>
    ```

- 위의 방법 외에 **@Bean**을 사용하여 등록하는 방법이 존재하지만 공식 문서에서  **mybatis-config.xml를 사용하는 방법을 권장한다.**

- 이후 Interceptor 클래스를 정의한다.

    ```java
    package com.example.common.interceptor;

    import java.util.Properties;

    import org.apache.ibatis.executor.Executor;
    import org.apache.ibatis.mapping.MappedStatement;
    import org.apache.ibatis.plugin.Interceptor;
    import org.apache.ibatis.plugin.Intercepts;
    import org.apache.ibatis.plugin.Invocation;
    import org.apache.ibatis.plugin.Plugin;
    import org.apache.ibatis.plugin.Signature;
    import org.apache.ibatis.session.ResultHandler;
    import org.apache.ibatis.session.RowBounds;

    @Intercepts({
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
        @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class})
    })
    public class MyMyBatisInterceptor implements Interceptor {

        @Override
        public Object intercept(Invocation invocation) throws Throwable {

            MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
            Object parameterObject = invocation.getArgs()[1];
            BoundSql boundSql = mappedStatement.getBoundSql(parameterObject);
            String sql = boundSql.getSql();

            // 쿼리 실행 직후 로그 출력
            logger.info("Executed SQL: {}", sql);
            logger.info("With Parameters: {}", parameterObject);

            // 실제 메서드 실행
            return invocation.proceed();
        }

        @Override
        public Object plugin(Object target) {
            return Plugin.wrap(target, this);
        }

        @Override
        public void setProperties(Properties properties) {
        }
    }
    ```

- **@Signature**에 대해

    - type : Interceptor가 가로챌 클래스 타입을 지정한다. Mybatis에서 적용가능한 대상 클래스는 아래와 같다.
        - Executor : 쿼리 실행과 트랜잭션 관리를 담당하는 인터페이스다.
        - StatementHandler : SQL 문을 준비하고 실행하기 직전의 단계에서 사용된다.
        - ParameterHandler : SQL에 바인딩되는 파라미터를 설정하는 역할을 담당한다.
        - ResultSetHandler : 쿼리 결과(ResultSet)를 객체로 변환하는 역할을 담당한다.

    - method : type으로 지정된 클래스에서 가로채고자 하는 메서드 이름이다. 
        - Executor.update : insert, update, delete 쿼리가 실행될 때 interceptor가 동작하도록 한다.
        - Executor.query : select 쿼리가 실행될 때 interceptor가 동작하도록 한다.
    
    - args : method에 선언된 메서드가 필요로 하는 파라미터를 대입한다.


### Mybatis Interceptor 사용 시의 장점

- Executor, StatementHandler 등의 메서드를 가로챌 수 있어, 쿼리 실행 단계 전반을 세밀하게 조정이 가능하다.

- 쿼리 실행 직전에 SQL을 변경하거나, 파라미터를 수정하는 등의 작업을 통해 다양한 커스터마이징이 가능하다.

- 특정 조건에 따라 특정 쿼리만 로깅, 즉 쿼리 필터링이 가능하다.