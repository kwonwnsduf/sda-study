# 집계 & NULL

## COUNT(*) vs COUNT(col)

| 구문         | 의미              |
| ---------- | --------------- |
| COUNT(*)   | **NULL 포함 행 수** |
| COUNT(col) | **NULL 제외**     |

## SUM/AVG + NULL
NULL은 계산에서 제외

단, 모든 값이 NULL이면 결과 NULL

## GROUP BY/ HAVING
WHERE → 집계 전

HAVING → 집계 후

ex)WHERE COUNT(*) > 1 X
   HAVING COUNT(*) > 1 O
   
## DISTINCT
> COUNT(DISTINCT col)

중복 제거 후 집계

# 서브쿼리 & JOIN 판단

## 서브쿼리 종류

### 스칼라 서브쿼리
결과 1행 1컬럼

SELECT 절에서 사용

> SELECT 
    booking_id, 
    show_name,
    (SELECT name FROM Users WHERE Users.id = Bookings.user_id) AS user_name
FROM Bookings;

### 다중행 서브쿼리
IN / EXISTS

> -- IN 사용 예시
SELECT price 
FROM Stocks 
WHERE stock_id IN (SELECT id FROM Categories WHERE name = 'IT');
## EXISTS vs IN
| 구분     | 특징        |
| ------ | --------- |
| IN     | 값 비교      |
| EXISTS | 존재 여부만 체크 |

NULL 있으면 IN 위험

## OUTER JOIN 조건 위치
X WHERE에 조건 → INNER JOIN처럼 됨

> -- 잘못된 예: 모든 유저를 보고 싶은데 예매 날짜가 2월인 사람만 남고 나머지는 증발함
SELECT U.name, B.show_name
FROM Users U
LEFT JOIN Bookings B ON U.id = B.user_id
WHERE B.booking_date >= '2026-02-01'; 
-- 결과: 2월에 예매 안 한 유저는 리스트에서 아예 사라짐 (INNER JOIN과 똑같아짐)

O ON에 조건 → OUTER 유지

> -- 올바른 예: 모든 유저를 보여주되, '2월 예매 건'만 옆에 붙여줘
SELECT U.name, B.show_name
FROM Users U
LEFT JOIN Bookings B ON U.id = B.user_id 
                   AND B.booking_date >= '2026-02-01';
                   
-- 결과: 2월 예매가 없는 유저도 이름은 나오고, 공연명만 NULL로 표시됨 (OUTER 유지)

## WHERE vs HAVING
| 구분     | 시점   |
| ------ | ---- |
| WHERE  | 그룹 전 |
| HAVING | 그룹 후 |
