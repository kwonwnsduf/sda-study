## 왜 RDS를 쓸까?
> [EC2]
 ├─ Spring Boot (Docker)
 └─ MySQL (Docker)

EC2 죽으면 DB도 같이 날아감

운영 환경에 절대 적합하지 않음

백업 / 장애 복구 / 스케일링 불가

> [EC2]
 └─ Spring Boot (Docker)

[AWS RDS]
 └─ MySQL (Managed DB)

 DB는 AWS가 관리
 EC2는 앱만 담당
 
##  AWS RDS란?

 MySQL / PostgreSQL / MariaDB / Oracle 지원

AWS가 대신 해줌:

백업

장애 복구

패치

모니터링

-> 우리는 DB 접속만 하면 됨

## RDS MySQL 생성

| 항목      | 설정                |
| ------- | ----------------- |
| 엔진      | MySQL             |
| 템플릿     | Free tier         |
| 퍼블릭 액세스 | **Yes**           |
| VPC     | EC2와 같은 VPC       |
| 보안그룹    | **EC2에서 3306 허용** |

EC2 → RDS 보안그룹 인바운드 허용

3306 포트

## EC2 ↔ RDS 네트워크 연결 구조
> EC2 (Security Group)
 └─ outbound: ALL 허용

RDS (Security Group)
 └─ inbound: MySQL 3306
      source: EC2 보안그룹

IP 직접 허용 

## RDS 접속 정보 정리
> 엔드포인트: xxxx.ap-northeast-2.rds.amazonaws.com
포트: 3306
DB명: sda_db
유저: admin
비밀번호: ****

RDS 생성 후 메모

## EC2에서 RDS 접속 테스트
> mysql -h RDS엔드포인트 -P 3306 -u admin -p

## Spring Boot → RDS 연결 설정
application-prod.yml

> spring:
  datasource:
    url: jdbc:mysql://RDS엔드포인트:3306/sda_db?serverTimezone=Asia/Seoul
    username: admin
    password: 비밀번호
    driver-class-name: com.mysql.cj.jdbc.Driver
jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

## docker-compose 수정 (DB 컨테이너 제거)
제거할 것 X
> mysql:
  image: mysql:8.0

최종구조
> services:
  app:
    image: sda-app
    environment:
      SPRING_PROFILES_ACTIVE: prod
    ports:
      - "8080:8080"
