---
title: Inner, Outer 조인의 이해
date: 2024-08-06 21:30:00 +09:00
published: true
toc_sticky: true
categories: [개발, DB]
tags: [Postgre, Spring Boot, Mybatis]
---

# 조인의 유형

- 아래와 같이 다앙한 join 존재

    ![조인의 종류](/assets/img/join-2/join.png)

## Inner 조인

- A 테이블에서 B 테이블로, B 테이블에서 A 테이블로 양쪽 방향성을 모두 가질 경우, Inner Join
        ![inner join](/assets/img/join-2/inner_join.png)


## Left Outer Join

- A 테이블에서 B 테이블로의 방향성만 가진 경우, Left Outer Join

- A 테이블의 레벨을 유지하는 조인
        ![left outer join](/assets/img/join-2/left_outer_join.png)

## Right Outer Join

- B 테이블에서 A 테이블로의 방향성만 가진 경우, Right Outer Join

- B 테이블의 레벨을 유지하는 조인

- Left Outer Join에 비해 잘 사용되지 않음
        ![right outer join](/assets/img/join-2/right_outer_join.png)

## Full Outer Join

- 배치 처리 시 사용되는 경우 존재

- A, B 양쪽 테이블의 레벨을 모두 유지하는 조인
        ![full_outer_join](/assets/img/join-2/full_outer_join.png)

### 실습 문제 풀기

#### 실습 환경에 대한 설명

- 이전 포스트인 Join, join-2의 테이블 사용

    - [조인 기반 메커니즘](2024-07-30-JOIN%20기반%20메커니즘.md)
    - [조인 기반 메커니즘 - 2](2024-08-05-JOIN%20기반%20메커니즘_2.md)

#### 실습 문제

- 주문이 단 한번도 없는 고객 정보 구하기.

    ```sql
    select *
    from nw.customers as cus
        left outer join nw.orders as orders on orders.customer_id = cus.customer_id 
    where orders.customer_id is null
    ```

- 부서정보와 부서에 소속된 직원명 정보 구하기. 부서가 직원을 가지고 있지 않더라도 부서정보는 표시되어야 함. 

    ```sql
    select dept.*, emp.ename 
    from hr.dept as dept
        left outer join hr.emp as emp on emp.deptno = dept.deptno 
    ```

- Madrid에 살고 있는 고객이 주문한 주문 정보를 구할것.
- 고객명, 주문id, 주문일자, 주문접수 직원명, 배송업체명을 구하되, 
- 만일 고객이 주문을 한번도 하지 않은 경우라도 고객정보는 빠지면 안됨. 이경우 주문 정보가 없으면 주문id를 0으로 나머지는 Null로 구할것. 

    ```sql
    select cus.contact_name , coalesce(orders.order_id, 0), orders.order_date, emp.first_name||' '||emp.last_name as emp_name, ship.company_name 
    from nw.customers as cus
        left outer join nw.orders as orders on orders.customer_id = cus.customer_id 
        left outer join nw.employees as emp on emp.employee_id = orders.employee_id 
        left outer join nw.shippers as ship on ship.shipper_id = orders.ship_via 
    where cus.city = 'Madrid'
    ```

- orders_items에 주문번호(order_id)가 없는 order_id를 가진 orders 데이터 찾기 

    ```sql
    select *
    from nw.orders as orders
        left outer join nw.order_items as ord_item on ord_item.order_id = orders.order_id 
    where ord_item.order_id is null
    ```

- orders 테이블에 없는 order_id가 있는 order_items 데이터 찾기. 

    ```sql
    select *
    from nw.order_items as ord_item
        left outer join nw.orders as orders on orders.order_id = ord_item.order_id 
    where ord_item.order_id is null
    ```

- 가장 마지막 두 문제는 결과값이 존재하지 않음

- DB에 컬럼이 제대로 생성되었는지 테스트하기 위해 사용하는 경우 존재