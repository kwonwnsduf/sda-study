#복합 인덱스
## 복합 인덱스란?
> CREATE INDEX idx_user_created
ON orders(user_id, created_at);

두 개를 묶어서 "하나의 인덱스"로 만든 것 

## 내부 구조
> (user_id, created_at) 기준 정렬
(1, 2024-01-01)
(1, 2024-01-05)
(1, 2024-02-01)
(2, 2024-01-03)
(2, 2024-01-20)
(3, 2024-02-10)

첫 번째 컬럼이 먼저 정렬

같은 user_id 안에서 created_at 정렬

## 컬럼순서는 왜 중요할까?
"복합 인덱스는 “왼쪽부터” 사용 가능"

> (user_id, created_at)

| WHERE 조건                       | 인덱스 사용                    |
| ------------------------------ | ------------------------- |
| user_id = 1                    | O                         |
| user_id = 1 AND created_at > ? | O                         |
| created_at > ?                 | X                         |
| created_at = ? AND user_id = ? | X (순서 무관해 보이지만 내부적으로 안 됨) |

## 잘못된 인덱스 예
쿼리
> SELECT *
FROM orders
WHERE user_id = 10
ORDER BY created_at DESC;

인덱스
> (created_at, user_id)

created_at 기준 정렬됨

user_id = 10 찾으려면 전체 훑어야 함

-> 풀 스캔 + 정렬 발생

올바른 인덱스
> (user_id, created_at)

	user_id로 먼저 좁힘

	그 안에서 created_at 정렬

	-> WHERE + ORDER BY 동시에 해결
    
## 복합 인덱스 vs 단일 인덱스 여러 개
상황
> WHERE user_id = ?
AND status = ?

인덱스 2개 X
> (user_id)
(status)

	DB가 하나만 선택

	나머지는 필터로 처리
    
복합 인덱스 1개
> (user_id, status)

	한 번에 탐색

	훨씬 빠름
    
## 커버링 인덱스
쿼리에 필요한 컬럼이 전부 인덱스에 있으면
테이블을 아예 안 읽는다.

예)
> SELECT user_id, created_at
FROM orders
WHERE user_id = 10;

> (user_id, created_at)
	
    인덱스만 읽고 끝
    
    테이블 접근 X
    
## 복합 인덱스 컬럼 순서 결정 공식
→ 범위 → 정렬

예)
> WHERE user_id = ?
AND status = ?
AND created_at >= ?
ORDER BY created_at DESC

최적 인덱스
> (user_id, status, created_at)
