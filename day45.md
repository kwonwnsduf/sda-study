# Dockerfile 정리
빌드와 런타임 분리(멀티 스테이지): 빌드 도구(Gradle/Maven) 포함 이미지를 운영에 올리지 않기

레이어 캐시 최적화: 자주 안 바뀌는 것(의존성 다운로드)을 앞에, 자주 바뀌는 소스는 뒤에

root로 실행하지 않기: 운영 컨테이너는 non-root 유저로

불필요 파일 제거: .dockerignore로 build/, .git/, node_modules/ 등 제외

고정 가능한 건 고정: base image 태그를 너무 “latest”로만 두지 않기 (운영은 버전 고정 추천)

헬스체크/포트/프로필 명확화: 컨테이너가 “살아있다” 기준을 운영에 맞게 정의

## 예시
 ---- 1) Build stage ----
FROM eclipse-temurin:17-jdk AS builder
WORKDIR /app

 캐시를 위해 Gradle 설정/래퍼 먼저 복사
COPY gradlew .
COPY gradle gradle
COPY build.gradle settings.gradle ./

의존성 캐시 (테스트는 빌드 단계에서만)
RUN ./gradlew dependencies --no-daemon || true

 소스 복사 후 빌드
COPY src src
RUN ./gradlew clean bootJar -x test --no-daemon

---- 2) Runtime stage ----
FROM eclipse-temurin:17-jre
WORKDIR /app

 보안: non-root 유저
RUN useradd -m appuser
USER appuser

jar 복사
COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8080
ENV JAVA_OPTS=""

 컨테이너 실행
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/app.jar"]

## .dockerignore

.git
.gradle
build
out
*.iml
.idea
*.log
node_modules

## 실수

COPY . .를 너무 앞에 둬서 캐시가 매번 깨짐

root로 실행 → 침해 시 피해 커짐

latest만 사용 → 어느 날 base image 바뀌며 장애

healthcheck 없음 → “컨테이너는 떴는데 서비스는 죽음” 상황 방치

# 환경변수

Secrets(민감정보): DB 비번, JWT secret, API key

환경 변수(.env): 서비스 설정값(포트, 프로필, URL 등)

앱 설정파일(application-prod.yml): 기본값/구조(민감정보는 X)

## .env 예시

SPRING_PROFILES_ACTIVE=local
DB_HOST=localhost
DB_PORT=3306
DB_NAME=ufc
DB_USER=root
DB_PASSWORD=localpw

SPRING_PROFILES_ACTIVE=prod
DB_HOST=your-rds-endpoint.amazonaws.com
DB_PORT=3306
DB_NAME=ufc
DB_USER=admin
DB_PASSWORD=****   # 실제는 secrets 추천
JWT_SECRET=****

.gitignore에 .env*넣는다

## secrets관리
EC2 서버에 .env.prod 직접 생성

GitHub Actions Secrets → 배포 시 서버에 echo로 주입

AWS SSM Parameter Store / Secrets Manager 사용

## Spring Boot에서 env를 연결하는 방식

application-prod.yml

spring:
  datasource:
    url: jdbc:mysql://${DB_HOST}:${DB_PORT}/${DB_NAME}?serverTimezone=UTC&characterEncoding=UTF-8
    username: ${DB_USER}
    password: ${DB_PASSWORD}

# Compose 운영

ex)services:
  app:
    image: kwonjunyeol/ufc-risk-platform:latest
    env_file:
      - .env.prod
    ports:
      - "8080:8080"
    restart: unless-stopped
    depends_on:
      - mysql
    networks:
      - app-net

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: ufc
      MYSQL_USER: app
      MYSQL_PASSWORD: apppw
      MYSQL_ROOT_PASSWORD: rootpw
    volumes:
      - mysql-data:/var/lib/mysql
    restart: unless-stopped
    networks:
      - app-net

networks:
  app-net:

volumes:
  mysql-data:

## restart

no : 기본값, 운영에선 비추천

always : 무조건 재시작 (의도치 않은 재시작도 계속)

unless-stopped : 운영에서 가장 흔한 선택 (내가 stop하면 안 켜짐)

on-failure[:N] : 비정상 종료 시만 재시작 (배치/워커에 적합)

## 네트워크 전략

같은 compose 안의 서비스들은 내부 DNS로 서비스명 접근 가능

예: app에서 DB 접근 host = mysql

외부 노출은 최소화

DB 컨테이너를 운영에서 직접 쓰면 ports로 외부 오픈하지 않는 편이 안전

(RDS 쓰면 DB 컨테이너 자체를 빼기도 함)

## 볼륨

DB: named volume 필수 (mysql-data)

로그:

간단: docker logs + log rotation

운영: 볼륨 마운트로 파일화 + 로그 수집(ELK/CloudWatch 등)

## 실수

.env.prod를 이미지에 넣음 → 보안사고

DB에 볼륨 안 걸고 운영 → 컨테이너 재생성 시 데이터 증발

depends_on 믿고 “DB 준비 완료”로 착각 → 실제론 “컨테이너 시작 순서”만 보장
→ 해결: app 쪽에서 재시도 로직 / healthcheck 사용

# Docker 장애 대응

A) 서비스 접속 안 됨

	docker ps (컨테이너 떠있나)

	docker logs -f <app> (부팅 실패 원인 확인)

	포트 충돌/방화벽/보안그룹 확인 (8080 열려있나)

B) 컨테이너는 떠있는데 API가 500

	앱 로그에서 DB 연결 에러/타임아웃 확인

	env 값(DB_HOST 등) 오타 확인

	RDS 보안그룹 인바운드: EC2 보안그룹을 소스로 허용했는지 확인

C) 디스크 부족 (매우 흔함)

	docker system df (docker가 디스크를 얼마나 쓰고 있는지 보여준다)

	docker image prune -f (안 쓰는 docker 이미지 정리)

	docker builder prune -f (도커 빌드 캐시 청소)

	정기적으로 docker image prune를 CI/CD 마지막에 넣기도 함
