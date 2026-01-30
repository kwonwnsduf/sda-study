## 바뀌는 구조

### before
> application-prod.yml
 └─ 비밀번호 직접 작성
 
### after
> application-prod.yml
 └─ 환경 변수 참조
      ↓
EC2 / Docker / AWS가 주입

## Spring Bott 설정변경
> spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}

## Docker Compose에서 환경 변수 주입
docker-compose.prod.yml
> services:
  app:
    image: sda-app
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_URL: jdbc:mysql://<RDS-ENDPOINT>:3306/sda
      DB_USER: admin
      DB_PASSWORD: ********

  
## EC2 서버에서 .env 파일로 분리
  
###  EC2에 .env 생성
>   nano .env

> DB_URL=jdbc:mysql://<RDS-ENDPOINT>:3306/sda
DB_USER=admin
DB_PASSWORD=비밀번호

### docker-compose.prod.yml
> env_file:
  - .env

.env는 절대 github에 올리면 안 된다
  
## 실행흐름
>  ./gradlew clean bootJar
docker build -t sda-app .
docker compose -f docker-compose.prod.yml up -d
