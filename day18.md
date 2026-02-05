# JOIN
JOIN = 여러 테이블을 하나의 결과처럼 보는 기술

## JOIN이 왜 필요할까?
테이블은 정규화 때문에 쪼개져 있다

예: members (회원)

	orders (주문)
 
주문 조회할 때 주문 정보 + 회원 이메일 같이 보고 싶으면?

->JOIN 필요

## JOIN 기본 구조
> SELECT 컬럼들
FROM 테이블A
JOIN 테이블B ON 연결조건;

예:
> SELECT *
FROM orders o
JOIN members m ON o.member_id = m.id;

orders와 members를 orders.member_id = members.id 기준으로 묶는다
    
## INNER JOIN(=JOIN)
양쪽 테이블에 모두 존재하는 데이터만
> SELECT o.id, m.email, o.price
FROM orders o
INNER JOIN members m ON o.member_id = m.id;

## LEFT JOIN
왼쪽 테이블은 전부 살리고, 오른쪽은 있으면 붙인다.
> SELECT *
FROM members m
LEFT JOIN orders o ON m.id = o.member_id;

## WHERE vs ON 차이
> SELECT *
FROM members m
LEFT JOIN orders o ON m.id = o.member_id
WHERE o.price > 10000;

->이 순간 LEFT JOIN이 INNER JOIN처럼 변함

올바른 예:
> SELECT *
FROM members m
LEFT JOIN orders o
  ON m.id = o.member_id
 AND o.price > 10000;

ON → JOIN 조건, "어떻게 붙일지"

WHERE → 결과 필터, "붙인 결과에서 뭘 걸러낼지"

## JOIN vs 서브쿼리
서브쿼리 예:
> SELECT *
FROM orders
WHERE member_id IN (
  SELECT id FROM members WHERE status = 'ACTIVE'
);

JOIN으로 바꾸면
> SELECT o.*
FROM orders o
JOIN members m ON o.member_id = m.id
WHERE m.status = 'ACTIVE';

## JOIN alias
> FROM orders o
JOIN members m

SQL 짧아짐

가독성 ↑

다중 JOIN에서 필수
