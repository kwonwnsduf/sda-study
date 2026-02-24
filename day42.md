## IAM

IAM = “누가(Identity) 무엇을 할 수 있는지(Access)”를 정하는 시스템

사람  ──(IAM User)──> AWS
서버  ──(IAM Role)──> AWS
규칙  ──(Policy)───> 허용 범위

사람과 서버는 계정이 다르다

## IAM User — “사람 계정”

IAM User = AWS 콘솔/CLI에 로그인하는 사람 계정

콘솔 로그인/Access Key 발급/ 직접 권한 부여

## IAM Role — “서버/서비스용 권한”

IAM Role = AWS 서비스가 ‘임시로’ 사용하는 권한

로그인x/ Access Key x/ EC2,Lambda,ECS 등에 붙임/ 임시 자격 증명 사용

S3 접근 권한이 있는 IAM Role 생성-> 그 Role을 EC2에 연결->EC2 안의 프로그램이 S3 호출

AWS가 자동으로: 임시 자격 증명 발급/ 권한 확인/ 만료되면 회수

### 왜 Role을 쓸까?

User 키를 서버에 넣으면 키 유출 위험, 키 만료/회전 어려움, 보안 사고

Role을 쓰면 키를 저장 안 함, AWS가 자동 발급/회수, 보안 좋음

## Policy — “권한 규칙서”

Policy = 무엇을 할 수 있는지 적은 규칙 문서

ex){
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}

Allow/ S3에서 파일 읽기/ my-bucket만 가능/ 호용한 것만

## 최소 권한 원칙

필요한 권한만 최소한으로 부여

나쁜 예) S3FullAccess
AdministratorAccess

ex) - s3:GetObject
- 특정 버킷만

## IAM User vs Role 비교

| 구분     | IAM User | IAM Role |
| ------ | -------- | -------- |
| 대상     | 사람       | 서버/서비스   |
| 로그인    | O        | X        |
| 키 보관   | O        | X        |
| 자격증명   | 고정       | 임시       |
| EC2 사용 | X        | O        |
| 보안     | 낮음       | 높음       |

EC2가 S3에 파일 업로드해야 함

방법1) 개발자가 IAM User 키를 EC2에 넣음-> 보안 사고

방법2) S3 권한 가진 IAM Role생성/ EC2에 Role 연결/ 코드에서 바로 S3 접근

*Accesss Key: AWS API를 호출하기 위한 비밀 열쇠
