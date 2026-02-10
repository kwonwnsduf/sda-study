#쿼리 튜닝
## 쿼리 튜닝이란?
같은 결과를 더 적은 비용(rows)으로 가져오게 만드는 것

목적: 읽는 row 수를 줄인다.

## 튜닝 절차
1. 느린 쿼리 발견
2. EXPLAIN으로 실행 계획 확인
3. 병목 원인 파악
4. 인덱스 or 쿼리 구조 수정
5. EXPLAIN 재확인

## 예제-WHERE 조건 튜니
상황
> SELECT *
FROM orders
WHERE user_id = 3
  AND status = 'PAID';

튜닝 전(인덱스 없음)
> EXPLAIN SELECT * FROM orders WHERE user_id = 3 AND status = 'PAID';

> type: ALL
key: NULL
rows: 1,200,000

문제점:
	테이블 전체 스캔

	user_id, status 조건 무시

	데이터 늘면 바로 장애	
    
튜닝: 복합 인덱스 추가
> CREATE INDEX idx_orders_user_status
ON orders(user_id, status);

튜닝 후 EXPLAIN
> type: ref
key: idx_orders_user_status
rows: 5

user_id → status 순서로 인덱스 탐색

1,200,000 → 5 rows

튜닝 성공

## 예제-ORDER BY 튜닝
상황:
> SELECT *
FROM orders
WHERE user_id = 3
ORDER BY created_at DESC
LIMIT 10;

튜닝 전
> EXPLAIN ...

> type: ref
key: idx_orders_user
rows: 12,000
Extra: Using filesort

문제점:
	user_id로는 찾았지만

	정렬은 메모리/디스크 정렬(filesort)
    
튜닝: 정렬까지 포함한 인덱스
> CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);

튜닝 후 EXPLAIN
> type: ref
key: idx_orders_user_created
rows: 10
Extra: Using index

해석
	인덱스 자체가 이미 정렬됨

	정렬 비용 제거

	LIMIT 10 → 딱 10건만 읽음
    
## 예제-COUNT(*) 튜닝
상황
> SELECT COUNT(*)
FROM orders
WHERE user_id = 3;

튜닝 전
> type: ALL
rows: 1,200,000

튜닝:커버링 인덱스 활용
> type: ALL
rows: 1,200,000

튜닝 후
> type: ref
key: idx_orders_user
Extra: Using index
rows: 2,000

테이블(row) 안 읽고 인덱스만 보고 COUNT

## 튜닝할 때 조심해야 할 것
인덱스 막 추가
	문제점:
    	쓰기 성능 하락

		인덱스 유지 비용 증가

SELECT *     
	튜닝 관점:
    	필요한 컬럼만 SELECT

		커버링 인덱스 깨짐
        
## 튜닝 판단 기준
| 상황     | 기준                |
| ------ | ----------------- |
| rows 수 | **줄어들었는가**        |
| type   | ALL → ref / range |
| key    | NULL → 실제 인덱스     |
| Extra  | filesort 제거       |
