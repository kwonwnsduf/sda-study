## 락이 왜 필요한가?
격리 수준(RR)은 읽기 일관성을 보장

하지만 쓰기 충돌(중복 차감, 중복 예약)은 자동으로 못 막음

->그래서 등장하는 게 락

“지금 이 데이터는 내가 처리 중이니, 남은 잠깐만 기다려라”

## 락의 큰 분류
### Shared Lock (읽기 락)
여러 트랜잭션이 동시에 읽기 가능

쓰기는 막음

### Exclusive Lock (쓰기 락)
한 트랜잭션만 접근 가능

다른 읽기/쓰기 전부 대기

InnoDB에서는 보통:

	UPDATE / DELETE → 쓰기 락

	SELECT ... FOR UPDATE → 쓰기 락
    
## Row Lock vs Table Lock
### Row Lock (행 단위 락)
특정 row만 잠금

동시성 좋다

> UPDATE seats SET status='RESERVED' WHERE id=10;

->id=10행만 락

### Table Lock
테이블 전체 잠금

동시성 안 좋다

> LOCK TABLE seats WRITE;

## 비관 락(Pessimistic Lock)
“충돌 날 것 같으니, 미리 잠가 놓자”

> SELECT * FROM seats WHERE id=10 FOR UPDATE;

### 동작
해당 row에 쓰기 락

다른 트랜잭션은:

	읽기 가능(일반 SELECT)

	수정 불가 (대기)

###  언제 쓰나?
좌석 예약, 재고 차감,쿠폰 1장

->한 명만 성공해야 하는 상황

## 낙관 락(Optimistic Lock)
“충돌은 드물다. 충돌 나면 실패시키자"

### 방식
row에 version 컬럼

UPDATE 시 버전 비교

> UPDATE seat
SET status='RESERVED', version=version+1
WHERE id=10 AND version=3;

영향 row = 1 → 성공

영향 row = 0 → 누가 먼저 수정 → 실패

### 언제 쓰나?
조회 많고 수정 적음

대기(블로킹) 싫을 때

## 데드락
서로가 서로의 락을 기다리며 영원히 멈춘 상태

예시
> T1: row A 락 → row B 대기
T2: row B 락 → row A 대기

DB는 감지 후 한 쪽을 강제 롤백

## 데드락이 실무에서 많이 나는 이유
### 락 순서 불일치
> T1: seat → payment
T2: payment → seat

### 범위 락 + 인덱스 없음
WHERE 조건이 인덱스 안 타면

의도치 않게 여러 row 락

### 트랜잭션이 너무 김
외부 API 호출

로그/이벤트 처리 포함

## 데드락 예방
### 락 순서 통일
ex)모든 코드에서 Seat → Reservation → Payment 고정

### 트랜잭션 짧게
검증 → 핵심 UPDATE만 트랜잭션

외부 호출은 트랜잭션 밖

### 인덱스 필수
> SELECT * FROM seat WHERE event_id=1 FOR UPDATE;

→ event_id 인덱스 없으면 범위 락 폭탄

### 데드락은 실패로 설계

## 비관 락 vs 낙관 락

| 기준    | 비관 락   | 낙관 락    |
| ----- | ------ | ------- |
| 충돌 빈도 | 높음     | 낮음      |
| 대기    | 있음     | 없음      |
| 실패    | 거의 없음  | 있음      |
| 사용처   | 좌석, 재고 | 좋아요, 수정 |
