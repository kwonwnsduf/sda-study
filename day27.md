#트랜잭션 격리 수준
## 트랜잭션 격리 수준이 왜 필요한가?
트랜잭션이 여러 개 동시에 실행되면 문제가 생김:

	다른 트랜잭션이 아직 커밋 안 한 데이터를 봄

	같은 SELECT를 두 번 했는데 결과가 달라짐

	갑자기 없던 행이 튀어나옴
    
    -> 이런 이상 현상을 어디까지 허용할지 정한 게 격리 수준
    
## 대표적인 이상 현상
### Dirty Read
커밋 안 된 데이터를 읽음

롤백되면 읽은 데이터가 거짓

### Non-Repeatable Read
같은 행을 두 번 읽었는데 값이 바뀜

(중간에 다른 트랜잭션이 UPDATE + COMMIT)

### Phantom Read
조건 조회 결과에 행 개수가 달라짐

(중간에 INSERT / DELETE)

## 격리 수준 4단계

| 격리 수준               | Dirty Read | Non-Repeatable | Phantom |
| ------------------- | ---------- | -------------- | ------- |
| READ UNCOMMITTED    |  허용       |  허용           |  허용    |
| READ COMMITTED  |  방지       |  허용           |  허용    |
| REPEATABLE READ |  방지       |  방지           |  허용    |
| SERIALIZABLE        |  방지       |  방지           |  방지    |

MySQL(InnoDB) 기본값 = REPEATABLE READ

## READ COMMITTED
커밋된 데이터만 읽는다

Dirty Read X

Non-Repeatable Read O

Phantom Read O

예시
> -- T1
SELECT balance FROM account WHERE id = 1; -- 100
-- T2
UPDATE account SET balance = 200 WHERE id = 1;
COMMIT;
-- T1
SELECT balance FROM account WHERE id = 1; -- 200 (값 바뀜!)

같은 트랜잭션인데 결과가 바뀜 → Non-Repeatable Read

## REPEATABLE READ
트랜잭션 시작 시점의 스냅샷을 끝까지 유지

같은 행을 몇 번 읽어도 값 고정

MySQL 기본 격리 수준

예시:
> -- T1 시작
SELECT balance FROM account WHERE id = 1; -- 100
-- T2
UPDATE account SET balance = 200 WHERE id = 1;
COMMIT;
-- T1
SELECT balance FROM account WHERE id = 1; -- 여전히 100

Non-Repeatable Read 방지

## Phantom Read
같은 조건으로 조회했는데 행 개수가 달라짐

예시:
> -- T1
SELECT * FROM orders WHERE price >= 10000; -- 3 rows
-- T2
INSERT INTO orders(price) VALUES (20000);
COMMIT;
-- T1
SELECT * FROM orders WHERE price >= 10000; -- 4 rows

새로운 행(Phantom) 등장

## ETC
REPEATABLE READ → Phantom 발생

MySQL(InnoDB):

	Next-Key Lock 때문에 대부분 방지됨

	하지만 완전 차단은 SERIALIZABLE
  
| 상황           | 권장 격리 수준          |
| ------------ | ----------------- |
| 일반 웹 서비스     | READ COMMITTED    |
| 금융 / 주문 / 결제 | REPEATABLE READ   |
| 완전 정합성       | SERIALIZABLE (비쌈) |

READ COMMITTED
→ 커밋된 것만 읽지만, 값은 바뀔 수 있다

REPEATABLE READ
→ 같은 행은 절대 안 바뀐다

Phantom Read
→ 조건 조회 결과에 행이 튀어나온다
