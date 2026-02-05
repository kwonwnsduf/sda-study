#기본 SELECT
## SELECT의 본질
> SELECT 컬럼
FROM 테이블
WHERE 조건;

"이 테이블에서, 이 조건을 만족하는, 이 컬럼만 가져와라"

## SELECT* vs 컬럼지정
> SELECT * FROM members;

모든 컬럼 조회

컬럼 추가되면 결과가 바뀜

> SELECT id, email, nickname
FROM members;

필요한 것만 가져와라

## WHERE 조건 필터링
기본 형태
> SELECT *
FROM members
WHERE email = 'test@test.com';

자주 쓰는 조건들
> WHERE id = 1
WHERE price >= 10000
WHERE status != 'DELETED'

여러 조건
> WHERE price >= 10000
  AND status = 'ACTIVE';

> WHERE status = 'ACTIVE'
   OR status = 'PENDING';

AND=둘 다 만족/ ON = 하나만 맍고

## NULL 조건
> WHERE deleted_at IS NULL;

> WHERE deleted_at IS NOT NULL;

## ORDER BY 정렬
오름차순
> SELECT *
FROM members
ORDER BY id;

내림차순
> ORDER BY id DESC;

여러 기준 정렬(먼저 쓴 컬럼이 1순위 기준)
> ORDER BY created_at DESC, id ASC;

## Limit/OFFSET- 개수 제한
최신 10개
> SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT 10;

페이징
> -- 1페이지
LIMIT 10 OFFSET 0;
-- 2페이지
LIMIT 10 OFFSET 10;
-- 3페이지
LIMIT 10 OFFSET 20;

공식:
> OFFSET = (페이지번호 - 1) × 페이지크기

## 실행순서
1. FROM
2. WHERE
3. SELECT
4. ORDER BY
5. LIMIT
