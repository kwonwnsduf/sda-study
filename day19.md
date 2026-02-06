# 서브쿼리
SQL 안에서 실행되는 또 다른 SQL
> SELECT *
FROM orders
WHERE user_id = (
    SELECT id
    FROM users
    WHERE email = 'a@test.com'
);

안쪽 SELECT = 서브쿼리

바깥 SELECT = 메인 쿼리

## 서브쿼리의 3가지 유형
### 스칼라 서브쿼리 (값 1개)
> SELECT name,
       (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;

하나의 값처럼 동작

SELECT / WHERE / ORDER BY 다 가능

2개 이상 나오면 에러

### 다중 행 서브쿼리 (여러 값)
결과가 여러 행
> SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
    WHERE total_price >= 100000
);

결과가 여러 개 → IN, ANY, ALL 필요

### 상관 서브쿼리 (바깥 쿼리랑 엮임)
> SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);

서브쿼리가 바깥 테이블(u) 을 참조

행 단위로 반복 실행

## IN vs EXISTS
### IN 서브쿼리
> SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);

동작 방식

	서브쿼리 먼저 실행

	결과 집합을 메모리에 저장

	바깥 쿼리에서 비교

특징

	결과 집합이 작을 때 좋음

	NULL 있으면 주의
    
### EXISTS 서브쿼리
> SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);

동작 방식

	users 한 행씩 읽음

	orders에 존재만 확인

	하나라도 있으면 즉시 TRUE

특징

	데이터 클수록 유리
	
	인덱스 있으면 매우 빠름

	NULL 문제 없음

| 상황         | 선택       |
| ---------- | -------- |
| 서브쿼리 결과 작다 | `IN`     |
| 테이블 크다     | `EXISTS` |
| 상관관계 필요    | `EXISTS` |
| 성능 중요      | `EXISTS` |

## JOIN vs 서브쿼리 
### JOIN
> SELECT DISTINCT u.*
FROM users u
JOIN orders o ON o.user_id = u.id;

DISTINCT로 같은 사용자(u.*)가 여러 번 나온 걸 1번으로 줄임

### EXISTS
> SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);

결과 동일

EXISTS쪽이 "의도"가 더 명확

## 서부쿼리 성능에서 중요한 것
인덱스 존재 여부
>CREATE INDEX idx_orders_user_id ON orders(user_id);

## 패턴
### 조건 만족 사용자 찾기
> WHERE EXISTS (SELECT 1 FROM orders ...)

### 통계용 스칼라 서브쿼리
> SELECT
  (SELECT COUNT(*) FROM orders) AS total_orders;

### 조건 필터용 서브쿼리
> WHERE id IN (SELECT ...)
