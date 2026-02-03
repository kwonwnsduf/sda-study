#운영환경분리&보안
## 환경분리
| 구분    | Local           | Production      |
| ----- | --------------- | --------------- |
| DB    | localhost MySQL | AWS RDS         |
| Redis | 로컬 컨테이너         | EC2/Elasticache |
| 비밀번호  | 테스트용            | 실제 운영용          |
| 로그    | 콘솔              | 파일/CloudWatch   |
| 접근    | 자유              | 최소 허용           |
## application.yml 정리
> spring:
  profiles:
    active: local

> spring:
  datasource:
    url: jdbc:mysql://localhost:3306/sda
    username: root
    password: 1234

> spring:
  datasource:
    url: jdbc:mysql://${DB_HOST}:3306/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    
## 환경변수
OS 또는 컨테이너 실행 시 주입되는 값/ 코드에 남지 않음
> DB_HOST=xxx.rds.amazonaws.com
DB_NAME=sda
DB_USER=admin
DB_PASSWORD=****

### docker compose에서 환경변수 사용
> services:
  app:
    image: sda-app
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_HOST: ${DB_HOST}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
### .env 설정
> DB_HOST=xxx.rds.amazonaws.com
DB_NAME=sda
DB_USER=admin
DB_PASSWORD=secret

.gitignore 필수

docker-compose.prod.yml에서 이렇게 사용
> services:
  app:
    env_file:
      - .env

## GitHub Actions Secrets
github->settings->secrets
등록 예:

DB_HOST

DB_NAME

DB_USER

DB_PASSWORD

> env:
  DB_HOST: ${{ secrets.DB_HOST }}
  DB_NAME: ${{ secrets.DB_NAME }}
  DB_USER: ${{ secrets.DB_USER }}
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}

.github/workflows/ci.yml
> name: CI
on:
  push:
    branches: [ "main" ]
jobs:
  build:
    runs-on: ubuntu-latest
    env:   #
      DB_HOST: ${{ secrets.DB_HOST }}
      DB_NAME: ${{ secrets.DB_NAME }}
      DB_USER: ${{ secrets.DB_USER }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
	 steps:
      - name: Checkout
        uses: actions/checkout@v4
	 - name: Build
        run: ./gradlew clean build

  
  
## AWS Security Group
### 방화벽 
누가/어디서/어떤 포트로 접근 가능?

### EC2 보안 그룹 예씨
| 포트       | 허용 대상     |
| -------- | --------- |
| 22 (SSH) | 내 IP만     |
| 8080     | 0.0.0.0/0 |
| 3306     | X (차단)    |

### RDS 보안 그룹 예시
| 포트   | 허용 대상     |
| ---- | --------- |
| 3306 | EC2 보안그룹만 |

RDS는 절대 0.0.0.0/0 열지 않는다

