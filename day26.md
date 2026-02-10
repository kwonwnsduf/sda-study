#트랜잭션 (ACID / Commit / Rollback)
## 트랜잭션이란?
트랜잭션 = 여러 SQL을 하나의 ‘작업 단위’로 묶는 것

전부 성공하면 전부 반영/하나라도 실패하면 전부 취소

## 왜 트랜잭션이 필요한가?
> UPDATE accounts SET balance = balance - 10000 WHERE id = 1;
UPDATE accounts SET balance = balance + 10000 WHERE id = 2;

중간에 서버 죽으면?

A 계좌: 돈 빠짐 

B 계좌: 돈 안 들어옴 

-> 돈 증발

트랜잭션 있을 때
> BEGIN;
UPDATE accounts SET balance = balance - 10000 WHERE id = 1;
UPDATE accounts SET balance = balance + 10000 WHERE id = 2;
COMMIT;

중간에 문제 생기면?
->ROLLBACK: 아무 일도 없었던 상태로 복구

## COMMIT/ROLLBACK
### COMMIT
지금까지 변경한 내용을

DB에 영구적으로 확정

### ROLLBACK
트랜잭션 시작 이후 변경한 내용

전부 취소

## ACID
### Atomicity(원자성)
하나라도 실패 → 전부 실패

중간 상태 없음

### Consistency (일관성)
규칙을 깨지 않는다

예:
	잔액은 음수가 되면 안 됨

	FK는 반드시 존재해야 함
    
트랜잭션 전/후 모두 규칙 만족

### Isolation (격리성)
동시에 실행돼도 서로 간섭하지 않음

다른 트랜잭션이 중간 결과를 보면 안 됨

### Durability (지속성)
COMMIT된 데이터는 안 사라짐

서버 다운돼도 디스크에 기록됨

## 트랜잭션의 범위
잘못된 예
> 컨트롤러 전체를 트랜잭션
→ 외부 API 호출 포함
→ 트랜잭션 너무 김

올바른 기준:
	“DB 상태가 함께 바뀌어야 하는 최소 단위”
    예: 
    	주문 상태 변경->결제 기록 저장->재고 감소 
        이 3개는 한 트랜잭션
        
## 자동 커밋 vs 수동 트랜잭션
### Auto Commit (기본값)
> UPDATE users SET name='A' WHERE id=1;

한 줄 실행, 바로 COMMIT, 되돌릴 수 없음

### 수동 트랜잭션
> BEGIN;
UPDATE ...
UPDATE ...
ROLLBACK;

## Spring/JPA에서 트랜잭션이 어떻게 연결되는가
@Transactional의 의미
> @Transactional
public void pay() {
    saveOrder();
    savePayment();
}

메서드 시작 → BEGIN

정상 종료 → COMMIT

예외 발생 → ROLLBACK
	RuntimeException → 롤백 O

	Checked Exception → 기본 롤백 X
    
## 트랜잭션이 있어도 생기는 문제
트랜잭션 ≠ 만능

	동시에 접근하면?

	누가 먼저 읽었는지?

	중간에 값이 바뀌면?
