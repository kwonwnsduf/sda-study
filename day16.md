#관계 설계(1:N/N:M))
## 관계란
관계=테이블과 테이블이 어떻게 연결되는가
 
 예: 회원 ↔ 주문
	
	게시글 ↔ 댓글

	사용자 ↔ 관심종목
 	
 이걸 숫자 관계로 표현하면:
 	
    1 : 1

	1 : N

	N : M
   
## 1:N 관계
개념: 하나가 여러 개를 가진다.

예:
	회원 1명 → 주문 여러 개

	게시글 1개 → 댓글 여러 개
    
설계 원칙: N쪽 테이블에 FK를 둔다

예:
> members
- id (PK)
- email

> orders
- id (PK)
- member_id (FK)
- price

> orders.member_id → members.id

왜 N쪽에 FK?
주문은 누가 한 주문인지 알아야 한다

회원 테이블에 주문 id를 여러 개 넣을 수 없음

->항상 많은 쪽이 외래키를 가진다.

## N:M 관계
개념: 여러개<->여러개

예:
	학생 ↔ 과목

	사용자 ↔ 관심종목

	주문 ↔ 상품

나쁜 예시:
> students
- subject_id1
- subject_id2

## N:M 해결법=조인 테이블
N:M은 반드시 중간 테이블로 풀어야 한다

예:
> users      watchlists      symbols
  1   ←→     N        ←→     1

테이블 구조
> users
- id (PK)

> symbols
- id (PK)

> watchlists
- id (PK)
- user_id (FK)
- symbol_id (FK)

예시:
> orders
- id

> products
- id

> order_products
- order_id (FK)
- product_id (FK)
- quantity
- price_at_order
