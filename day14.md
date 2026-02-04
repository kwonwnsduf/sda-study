#테이블 설계 기본
## Pk-테이블의 정체성
각 row를 유일하게 식별하는 값

중복 X / NULL X

테이블의 “주민등록번호”

> id BIGINT PRIMARY KEY

왜 PK가 중요한가?

 JOIN 기준점

 인덱스 자동 생성 (성능)

 FK가 반드시 참조하는 대상
 
## FK (Foreign Key) — 관계의 연결고리
FK란?

다른 테이블의 PK를 참조

“이 데이터는 저 데이터에 속한다”

> member_id BIGINT REFERENCES members(id)

FK가 주는 것

	데이터 무결성

		없는 member_id 저장 X

관계가 DB 레벨에서 보장

## NOT NULL / UNIQUE — 제약 조건의 의미
### NOT NULL
> email VARCHAR(100) NOT NULL

반드시 값이 있어야 함

“비즈니스 필수값”

### UNIQUE
> email VARCHAR(100) UNIQUE

중복 불가

로그인 ID, 주민번호 같은 개념

## 데이터 타입 선택 기준

### 문자열
| 타입      | 언제 쓰나             |
| ------- | ----------------- |
| VARCHAR | 대부분               |
| CHAR    | 고정 길이 (거의 안 씀)    |
| TEXT    | 길이 예측 불가 (설명, 내용) |

### 숫자
| 타입           | 용도                |
| ------------ | ----------------- |
| BIGINT       | PK, FK            |
| INT          | 카운트, 상태값          |
| DECIMAL      | **돈, 비율 (★★★★★)** |
| FLOAT/DOUBLE | X 금융 데이터 금지       |

### 날짜
| 타입        | 용도           |
| --------- | ------------ |
| DATE      | 날짜만          |
| DATETIME  | 날짜 + 시간      |
| TIMESTAMP | 시간 + 타임존(주의) |

## 좋은 테이블의 최소 조건

 PK 존재
 의미 없는 surrogate key 사용 (id)
 FK로 관계 표현
 NOT NULL은 진짜 필수만
 타입은 미래를 고려

## 예시
> CREATE TABLE members (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(100) NOT NULL UNIQUE,
  created_at DATETIME NOT NULL
);

> CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  member_id BIGINT NOT NULL,
  total_price DECIMAL(10,2) NOT NULL,
  created_at DATETIME NOT NULL,
  FOREIGN KEY (member_id) REFERENCES members(id)
);
