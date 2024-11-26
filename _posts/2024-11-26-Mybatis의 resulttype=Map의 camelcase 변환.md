---
title: Mybatis의 resultType = Map의 camelcase 적용 
date: 2024-11-26 19:30:00 +09:00
published: true
toc_sticky: true
categories: [개발, 성능 개선]
tags: [Spring Boot]
---

# mapUnderscoreToCamelCase의 사용 이유

- mybatis를 사용하여 DB와 통신 시 실제 table의 컬럼명이 emp_name 이런 식으로 스네이크 케이스인 경우가 존재한다.

- 스네이크 케이스로된 칼럼명에 맞춰 DTO의 변수명도 **"String emp_name"**이라고 할 경우, 자바의 기본적인 카멜 케이스 형태랑 맞지 않는다.

- 이런 문제를 해결하기 위해 DTO로 받아올 경우, 해당 DTO 내 변수명에 맞게 카멜 케이스로 변환하여 매핑시켜주는 옵션이 **"mapUnderscoreToCamelCase"** 옵션이다.

- mapUnderscoreToCamelCase 옵션을 사용하기 위해서는 mybatis-config.xml의 settings 태그 내에 아래의 옵션을 선언하여 사용할 수 있다.
    ```xml
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    ```


## mapUnderscoreToCamelCase의 한계

- 상황에 따라 DTO를 선언하여 데이터를 받아오고 싶지만 <span style="color:red">**resultType="Map"**</span> 처럼 사용해야 하는 경우가 존재한다.

- 이 경우에는 mapUnderscoreToCamelCase 옵션이 true로 선언되어 있어도 emp_name를 받아올 경우, **key값이 empName이 아닌 emp_name이 된다.**


### 반환타입이 Map일 때, mapUnderscoreToCamelCase가 적용되지 않는 이유

- 단순 SELECT 쿼리를 디버깅하여 따라갈 경우, 객체에 대한 접근과 수정 기능을 제공하는 클래스인 **MetaObject**와 MetaObejct 생성자 내에서 실제 반환 타입에 전달될 값이 저장된 **originalObject**, originalObject와 반환객체의 매핑을 수행하는 **Wrapper**클래스를 확인할 수 있다.
    ```java
    // org.apache.ibatis.reflection.MetaObject 경로에 존재
    public class MetaObject {

        private final Object originalObject;
        private final ObjectWrapper objectWrapper;
        private final ObjectFactory objectFactory;
        private final ObjectWrapperFactory objectWrapperFactory;
        private final ReflectorFactory reflectorFactory;

        private MetaObject(Object object, ObjectFactory objectFactory, ObjectWrapperFactory objectWrapperFactory,
            ReflectorFactory reflectorFactory) {
        ...
            if (object instanceof ObjectWrapper) {
            this.objectWrapper = (ObjectWrapper) object;
            } else if (objectWrapperFactory.hasWrapperFor(object)) {
            this.objectWrapper = objectWrapperFactory.getWrapperFor(this, object);
            } else if (object instanceof Map) {
            this.objectWrapper = new MapWrapper(this, (Map) object);
            } else if (object instanceof Collection) {
            this.objectWrapper = new CollectionWrapper(this, (Collection) object);
            } else {
            this.objectWrapper = new BeanWrapper(this, object);
            }
        }
        ...
    }
    ```

- 위의 코드를 보면 반환타입이 Map일 경우 <span style="color:red">**MapWrapper**</span>를, DTO의 경우 <span style="color:red">**BeanWrapper**</span>를 따라가게 된다.

- MapWrapper, BeanWrapper 모두 **findProperty** 메소드를 호출하여 key값으로 사용할 변수명을 반환하게 된다.

- 먼저 MapWpper의 findProperty는 **useCamelCaseMapping**을 파라미터로 받지만, 그 값을 확인하지 않고 name을 반환한다.
    ```java
    // org.apache.ibatis.reflection.wrapper.MapWrapper 경로에 존재
    @Override
    public String findProperty(String name, boolean useCamelCaseMapping) {
        return name;
    }
    ```

- 반면 BeanWrapper는 **metaClass.findProperty()** 메소드를 활용하여 useCamelCaseMapping의 값에 따라 카멜 케이스로 변환하여 반환한다.
    ```java
    // org.apache.ibatis.reflection.wrapper.BeanWrapper 경로에 존재
    // BeanWrapper의 findProperty
    @Override
    public String findProperty(String name, boolean useCamelCaseMapping) {
        return metaClass.findProperty(name, useCamelCaseMapping);
    }
    ...

    // org.apache.ibatis.reflection.MetaClass 경로에 존재
    // MetaClass의 findProperty 
    public String findProperty(String name, boolean useCamelCaseMapping) {
        if (useCamelCaseMapping) {
        name = name.replace("_", "");
        }
        return findProperty(name);
    }
    ```


## 반환타입이 Map일 때, camelcase 적용하기

- 따라서, **resultType=Map**일 때 자동으로 카멜 케이스로 변환하기 위해선 별도의 설정이 필요하다.

- 아래는 커스텀 Wrapper 구현 및 적용하여 해당 문제를 해결하는 방법이다.
 

### 커스텀 Wrapper 구현 및 적용

1. Custom MapWrapper 구현
    - MapWrapper를 상속받아 키를 camelCase로 변환하는 로직을 추가한다.
        ```java
            import org.apache.ibatis.reflection.MetaObject;
            import org.apache.ibatis.reflection.wrapper.MapWrapper;

            import java.util.HashMap;
            import java.util.Map;

            public class CustomMapWrapper extends MapWrapper {
                public CustomMapWrapper(MetaObject metaObject, Map<String, Object> map) {
                    super(metaObject, map);
                }

                @Override
                public String findProperty(String name, boolean useCamelCaseMapping) {
                    if (useCamelCaseMapping && name != null) {
                        StringBuilder result = new StringBuilder();
                        boolean upperCase = false;
                        for (int i = 0; i < name.length(); i++) {
                            char c = name.charAt(i);
                            if (c == '_') {
                                upperCase = true;
                            } else if (upperCase) {
                                result.append(Character.toUpperCase(c));
                                upperCase = false;
                            } else {
                                result.append(c);
                            }
                        }
                        return result.toString();
                    }
                    return name;
                }
            }
        ```

2. Custom ObjectWrapperFactory 구현
    - MyBatis에서 기본 Wrapper 대신 커스텀 Wrapper를 사용할 수 있도록 설정하기 위한 CustomObjectWrapperFactory을 구현한다.
        ```java
            import org.apache.ibatis.reflection.MetaObject;
            import org.apache.ibatis.reflection.factory.ObjectFactory;
            import org.apache.ibatis.reflection.wrapper.DefaultObjectWrapperFactory;
            import org.apache.ibatis.reflection.wrapper.ObjectWrapper;

            import java.util.Map;

            public class CustomObjectWrapperFactory extends DefaultObjectWrapperFactory {
                @Override
                public boolean hasWrapperFor(Object object) {
                    return object instanceof Map;
                }

                @Override
                public ObjectWrapper getWrapperFor(MetaObject metaObject, Object object) {
                    if (object instanceof Map) {
                        return new CustomMapWrapper(metaObject, (Map<String, Object>) object);
                    }
                    return super.getWrapperFor(metaObject, object);
                }
            }
        ```

3. MyBatis Configuration에 CustomObjectWrapperFactory 등록
    - mybatis-config.xml에 CustomObjectWrapperFactory를 등록한다.
        ```xml
        ...
        <configuration>
            <objectWrapperFactory type="com.example.CustomObjectWrapperFactory"/>
        </configuration>
        ...
        ```

- 위의 방법을 통해 반환타입이 Map이어도 카멜 케이스로 변환되어 매핑시킬 수 있으며 필요에 따라 추가적인 WrapperFactory를 만들어 다른 커스텀 Wrapper를 구현, 적용할 수 있다.