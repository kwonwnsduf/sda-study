#SQL 패턴
## 통계 쿼리
### 기본집계
> SELECT COUNT(*) FROM orders;
SELECT SUM(total_price) FROM orders;
SELECT AVG(total_price) FROM orders;

### 그룹별 통계
> SELECT user_id,
       COUNT(*) AS order_count,
       SUM(total_price) AS total_amount
FROM orders
GROUP BY user_id;

->회원별 주문 횟수 + 총 구매금액

### 조건 있는 통계
> SELECT user_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 3;

WHERE->row 필터링
HAVING->그룹 필터링

## 랭킹/순위 쿼리
### 단순 랭킹
> SELECT user_id, SUM(total_price) AS total_amount
FROM orders
GROUP BY user_id
ORDER BY total_amount DESC;

"누가 제일 많이 샀는지"

### 상위 N명
> SELECT user_id, SUM(total_price) AS total_amount
FROM orders
GROUP BY user_id
ORDER BY total_amount DESC
LIMIT 10;

### 순위 번호 붙이기
> SELECT user_id,
       SUM(total_price) AS total_amount,
       RANK() OVER (ORDER BY SUM(total_price) DESC) AS ranking
FROM orders
GROUP BY user_id;

## 페이징
### X OFFSET 페이징
> SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 1000;

문제점 
	OFFSET이 커질수록 점점 느려짐

	데이터 추가/삭제 시 중복 or 누락 발생

	대용량 테이블에서 치명적
    
### 커서 기반 페이징
> SELECT *
FROM orders
WHERE created_at < '2026-02-01 12:00:00'
ORDER BY created_at DESC
LIMIT 20;

'2026-02-01 12:00:00' 이게 커서(cursor)

## ETC
### 존재 여부 판단
> SELECT EXISTS (
    SELECT 1 FROM orders WHERE user_id = 3
);

->boolean 체크용

### 최신 데이터 1건
> SELECT *
FROM orders
WHERE user_id = 3
ORDER BY created_at DESC
LIMIT 1;

### 통계+조건 결함
> SELECT COUNT(*)
FROM orders
WHERE status = 'SUCCESS'
  AND created_at >= '2026-01-01';
