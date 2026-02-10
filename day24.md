#EXPLAIN
## EXPLAIN이란?
> EXPLAIN SELECT * FROM orders WHERE user_id = 3;

이건 SQL을 실행하는 게 아니라 “이 쿼리를 어떻게 실행할 건지 계획 보여줘”를 뜻한다.

즉, DB의 내부 실행 전략을 보는 명령어

## EXPLAIN 결과
 | 컬럼       | 의미        | 왜 중요      |
| -------- | --------- | --------- |
| **type** | 접근 방식     | 성능 1순위    |
| **key**  | 사용한 인덱스   | 인덱스 적용 여부 |
| **rows** | 읽을 예상 행 수 | 비용 추정     |

## type
type = “DB가 데이터를 찾는 방식”

성능 좋은 순서
| type  | 의미       | 해석   |
| ----- | -------- | ---- |
| const | PK 1건 조회 | 최고   |
| ref   | 인덱스 사용   | 좋음   |
| range | 범위 조회    | 괜찮   |
| index | 인덱스 풀스캔  | 위험   |
| ALL   | 테이블 풀스캔  | X 최악 |

예시1-좋은경우
> EXPLAIN SELECT * FROM users WHERE id = 1;

> type: const
key: PRIMARY
rows: 1

의미:
	PK로 바로 찾음

	딱 1행만 읽음

	이상적인 쿼리

예시2-위험한 경우
> EXPLAIN SELECT * FROM orders WHERE price > 10000;

> type: ALL
key: NULL
rows: 1,200,000

의미:

	인덱스 없음

	테이블 전체 다 읽음

	데이터 늘면 바로 장애
    
## key-"실제로 사용된 인덱스"
인덱스가 있다고 쓰는 게 아님

사용된 인덱스만 나온다

예시
> EXPLAIN SELECT * FROM orders WHERE user_id = 3;

> EXPLAIN SELECT * FROM orders WHERE user_id = 3;

	user_id 인덱스를 실제로 사용

	인덱스 설계가 성공했다는 뜻
    
> WHERE DATE(created_at) = '2024-01-01'

> key: NULL

	함수 씌우면 인덱스 못 씀

	created_at 인덱스 있어도 무시됨
    
## rows-"얼마나 많이 읽을까?"
rows는 실제 행 수 X, DB가 예상하는 행 수

> rows: 1000000

의미:
	“대충 이 정도 읽어야 할 것 같아”

	많을수록 느림
    
비교 예시:

튜닝 전
> type: ALL
rows: 1,000,000

인덱스 추가 후
> type: ref
rows: 12
