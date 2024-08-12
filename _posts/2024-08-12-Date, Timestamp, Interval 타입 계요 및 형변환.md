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


