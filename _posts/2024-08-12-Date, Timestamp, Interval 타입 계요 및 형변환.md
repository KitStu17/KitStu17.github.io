---
title: Date, Timestamp, Interval 타입 개요 및 형변환
date: 2024-08-12 21:30:00 +09:00
published: true
toc_sticky: true
categories: [개발, DB]
tags: [Postgre, Spring Boot, Mybatis]
---

# 각 타입의 개요

- Date : 일자로서 년, 월, 일 정보를 가짐(YYYY-MM-DD)

- Timestamp : Date + 시간 정보를 가짐(YYYY-MM-DD HH24:MI:SS)

- Time : 오직 시간 정보만 가짐(HH24:MI:SS)

- Interval : N days HH24:MI_SS


## 문자열을 Date, Timestamp로 변환

- TO_DATE('2024-08-12', 'YYYY-MM-DD') => 2024-08-12

- TO_TIMESTAMP('2024-08-12', 'YYYY-MM-DD') => 2024-08-12 00:00:00.000 +0900

- TO_TIMESTAMP('2024-08-12 19:56:52', 'YYYY-MM-DD HH24:MI:SS') => 2024-08-12 19:56:52.000 +0900

- Date, Timestamp 모두 첫 파라미터는 변환할 문자열, 다음 파라미터는 년, 월, 일의 구성을 알려주는 포맷팅이다.


## Date, Timestamp를 문자열로 변환

- TO_CHAR(CURRENT_DATE, 'YYYY-MM-DD') => 2024-08-12

### 포맷팅 패턴

- 아래의 그림과 같은 포맷팅을 가진다.

    ![포맷팅 패턴](/assets/img/typecast/fommet.png)


### ::date, ::timestamp, ::text를 사용한 형 변환

- ::date, ::timestamp 등을 이용하여 간단한 형 변환 가능

- date -> timestamp
    - select to_date('2024-08-12', 'yyyy-mm-dd')::timestamp;

- timestamp -> text
    - select to_timestamp('2024-08-12', 'yyyy-mm-dd')::text;


### 날짜와 시간 연산 - Interval 연산

- Date 타입에 숫자를 더하거나 빼면 해당하는 일자를 더하거나 빼서 반환

- Timestamp 타입에 숫자를 더하거나 빼면 오류 발생
    - Timestamp 타입은 Interval을 사용하여 연산
    - SELECT TO_TIMESTAMP('2024-08-12', 'YYYY-MM-DD') - INTERVAL'3'DAY AS calc_date

- Date 타입에 Interval을 사용하여 연산할 경우, Timestamp 타입으로 반환됨
    ```sql
    select to_date('2024-08-12', 'yyyy-mm-dd') - to_date('2024-01-01', 'yyyy-mm-dd') as interval_01
	, pg_typeof(to_date('2024-08-12', 'yyyy-mm-dd') - to_date('2024-01-01', 'yyyy-mm-dd')) as type ;
    ```

- Date - Date는 정수형으로, Timestamp - Timestamp는 Interval로 출력됨
    ```sql
    select to_timestamp('2024-08-12 19:00:00', 'yyyy-mm-dd hh24:mi:ss') 
     - to_timestamp('2024-01-01 12:00:00', 'yyyy-mm-dd hh24:mi:ss') as time_01
     , pg_typeof(to_timestamp('2024-08-12 19:00:00', 'yyyy-mm-dd hh24:mi:ss') 
     - to_timestamp('2024-01-01 12:00:00', 'yyyy-mm-dd hh24:mi:ss')) as type ;
    ```

- justify_interval(), age()를 통해 Timestamp - Date 연산도 가능
    - 단, justify_interval()은 **한 달을 무조건 30일**로 계산