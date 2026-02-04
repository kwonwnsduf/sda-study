#SQL 전체 구조 이해
## RDBMS 개념
### RDBMS란?
Relational DataBase Management System

 데이터를 표(Table) 형태로 저장

 표들 사이를 관계(Relation) 로 연결

 SQL로 데이터 조작
 
### 왜 RDBMS를 쓰나?
 데이터 정합성 (틀린 데이터 방지)

 관계 기반 조회 (JOIN)

 트랜잭션 보장 (은행, 주문, 결제)

## 테이블/Row/Column
### 테이블
 하나의 개념

 예: users, orders, decisions
 
### Row (행)
 데이터 1건

 user 1명, 주문 1건
 
### Column (열)
속성

id, email, created_at

| id | email                     | created_at |
| -- | ------------------------- | ---------- |
| 1  | [a@a.com](mailto:a@a.com) | 2026-02-01 |

## SQL 실행흐름
> SELECT *
FROM users
WHERE id = 1;

1.파싱
→ 문법 맞는지

2.최적화
→ 인덱스 쓸지 판단

3.실행 계획 생성

4.스토리지 엔진 접근

5.Row 조회 → 결과 반환
