## AWS는 무엇을 제공하는가?
AWS = 전 세계에 분산된 데이터센터를 내가 원하는 만큼, 필요한 서비스 단위로, 즉시 빌려 쓸 수 있게 만든 플랫폼

## Region/AZ
### Region

지리적으로 떨어진 큰 묶음

ex) ap-northeast-2 → 서울

	ap-northeast-1 → 도쿄

	us-east-1 → 버지니아
 
리전 간 데이터는 기본적으로 분리

법/지연시간/재해 대응 목적

서비스 생성 시 반드시 리전 선택

### AZ
하나의 리전 안에 있는 독립 데이터센터

보통 2~4개 존재 (서울 리전: ap-northeast-2a / 2b / 2c 등)

전기, 네트워크, 건물까지 완전히 분리

AZ 하나가 터져도 다른 AZ는 정상

| 구분     | 의미                |
| ------ | ----------------- |
| Region | 재해 단위 (지진, 국가 장애) |
| AZ     | 장애 단위 (전원, 네트워크)  |

## AWS 서비스 분류

### Compute

CPU / RAM을 빌려주는 영역

	EC2 → 가상 서버

	ECS / EKS → 컨테이너 실행
    
    	ECS: AWS 전용 컨테이너 서비스
        EKS: K8s 기반

	Lambda → 서버 없는 함수 실행
    
### Storage

데이터를 어디에 어떻게 저장할 것인가

	EBS → EC2에 붙는 디스크

	S3 → 객체 스토리지 (파일) --서버랑 완전히 독립/ 매우 큰 용량/ 웹에서도 바로 접근 가능

	EFS → 공유 파일 시스템

EBS = 서버 종속

S3 = 서버 독립

### Database

데이터 구조 + 트랜잭션 관리

	RDS → MySQL / PostgreSQL 등

	DynamoDB → NoSQL(Key-Value 기반)

	Aurora → AWS 최적화 RDB
    
### Network

통신을 통제하는 영역

	VPC: AWS 안의 가상 네트워크

	Subnet: 네트워크 분할

	Route Table: 길 안내

	IGW: 인터넷 연결
    
    NAT: 내부 -> 외부만 허용
    
### Security / Identity

누가 무엇을 할 수 있는가

	IAM User: 사람 계정

	IAM Role: 서비스용 권한
    		  ex) EC2->S3 접근 

	Policy: 권한 규칙 문서(JSON)

### Monitoring / Management

보고, 기록하고, 자동화

	CloudWatch: CPU, 메모리, 로그

	CloudTrail: 누가 언제 무엇을 했는지

	Auto Scaling: 트래픽 많으면 서버 늘리고 적으면 줄인다
    
## 왜 이렇게 나눠 놨을까?

과거 서버 하나에:OS,DB,파일,보안,백업
→ 한 곳 고장 = 전체 장애


| 영역   | 책임 분리 |
| ---- | ----- |
| 서버   | EC2   |
| 디스크  | EBS   |
| DB   | RDS   |
| 파일   | S3    |
| 네트워크 | VPC   |
| 권한   | IAM   |

