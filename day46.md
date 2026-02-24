# Nginx란

Nginx는 서버 앞단에 서서 요청을 대신 받아 처리해주는 ‘관문 서버’다.

[ 사용자 ]
    ↓
[ Nginx ]   ← 운영의 핵심
    ↓
[ Spring Boot ]

Spring Boot는 비즈니스 로직에 집중
Nginx는 트래픽, 보안, 배포 안정성을 담당

## Reverse Proxy

| 구분 | Proxy   | Reverse Proxy |
| -- | ------- | ------------- |
| 위치 | 사용자 쪽   | 서버 쪽          |
| 대상 | 외부 서버   | 내부 서버         |
| 목적 | 우회, 익명성 | 보호, 제어, 안정성   |

Reverse Proxy가 하는 일:

	사용자는 Spring Boot의 존재를 모름

	Nginx가 요청을 받아 적절한 서버로 전달
    
    Spring Boot를 직접 노출 X, Nginx 뒤에 숨김 O
    
## 포트 숨기기

왜 포트를 숨겨야 할까?

	포트가 노출되면 직접 공격 가능, 우회 접근 가능, 운영 통제 불가
    
외부 사용자: 80 / 443
      ↓
   Nginx
      ↓
Spring Boot : 8080 (내부 전용)

외부에서는 80/443만 접근 가능
8080은 서버 내부에서만 사용

Spring Boot를 80으로 실행하면 안 좋은 이유:

	80/443 → root 권한 필요 (보안 위험)

	HTTPS 처리 불가능

	무중단 배포 불가능
	
	여러 서비스 공존 불가
 
Nginx (80/443)
Spring Boot (8080)

# Nginx (Docker+ Spring Boot 연결)

> 브라우저
  ↓  http://localhost
Nginx (80)
  ↓  proxy_pass
Spring Boot (8080)

## 왜 docker?

Nginx, Spring Boot를 각각 하나의 상자(container) 로 분리

포트 / 네트워크 / 역할이 눈에 보이게 됨

운영 구조랑 거의 동일

## Spring boot

application.yml 
ex) server:
  port: 8080
  
테스트용 컨트롤러
 ex)@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Boot";
    }
}

Dockerfile
ex) FROM eclipse-temurin:17-jre
WORKDIR /app
COPY build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]

## Nginx 설정 파일

nignx.conf 
ex)server {
    listen 80;

    location / {
        proxy_pass http://spring:8080;
    }
}

spring → docker-compose에서 정의한 서비스 이름

8080 → Spring Boot 내부 포트

docker-compose.yml
ex)version: "3.8"

services:
  spring:
    build: .
    container_name: spring-app
    expose:
      - "8080"

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - spring

## 실행

docker-compose up -d -> http://localhost

	브라우저 → localhost:80

	Docker → nginx 컨테이너

	Nginx → proxy_pass

	spring 컨테이너 8080

	Spring Boot 응답

	다시 Nginx → 브라우저

# HTTPS 개념

## HTTP vs HTTPS

HTTP(80): 브라우저(클라이언트)와 서버가 “요청/응답”을 주고받는 규칙(프로토콜)
	  평문(plain text) 통신이 기본
		→ 중간에서 가로채면 내용을 그대로 읽을 수 있음
	  HTTP 자체는 “누가 진짜 서버인지” 증명해주지 않음

HTTPS(443): HTTPS = HTTP + (TLS로 보안 기능 추가)
	   
       기밀성(Confidentiality): 내용이 암호화돼서 중간에서 봐도 해독 불가

	   무결성(Integrity):전송 중 내용이 바뀌면 탐지 가능 (위/변조 방지)

	  인증(Authentication): 내가 접속한 서버가 진짜 그 도메인의 서버인지 확인, 인증서
   
## HTTPS에서 문제

“이 서버, 진짜 맞아?”

암호화만 하면 끝이 아니다. ‘가짜 서버’와 암호화해도 소용없다. 

->그래서 필요한게 "인증서"

## 인증서

인증서 = “이 서버는 진짜입니다” 증명서

인증서 안에는 도메인 이름, 공개키 발급자 정보(CA), 유효기간

CA: 이 서버가 진짜 맞다고 보증해주는 공식 기관(ex: Let's Encrypt)

## HTTPS가 만들어지는 과정
> 1. 브라우저: https://도메인 접속
2. 서버: 인증서 보내줌
3. 브라우저: CA 서명 확인 (진짜인지 검사)
4. OK면:
   - 암호화 키 생성
   - 이후 통신은 전부 암호화

## Nginx가 HTTPS 처리하면?

> 브라우저 ==HTTPS==> Nginx ==HTTP==> Spring Boot

인증서 관리 → Nginx

Spring Boot는 평문 HTTP만 처리

브라우저
  ↓  https://localhost (443)
Nginx (443 )
  ↓  내부 HTTP
Spring Boot (8080)

# Nginx(Let's Encrypt + certbot)

> 브라우저
  ↓  https://도메인 (443 )
Nginx
  ├─ 인증서 제공 (Let’s Encrypt)
  ├─ TLS Handshake
  ↓
Spring Boot (8080, 내부 HTTP)

## Let’s Encrypt & certbot

### Let’s Encrypt

무료 인증서 발급 기관(CA)

전 세계 브라우저가 신뢰함

90일짜리 인증서

### certbot

Let’s Encrypt에서 인증서를
자동으로 발급·갱신해주는 도구

사람 대신 서버가 알아서 처리

## Nginx 기본 HTTP 설정

nginx.conf (HTTPS 붙이기 전 단계)

ex) server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://spring:8080;
    }
}

## certbot 설치 (EC2)

sudo apt update
sudo apt install certbot python3-certbot-nginx -y

python3-certbot-nginx : Nginx 자동 연동 플러그인

## 인증서 발급

sudo certbot --nginx -d api.example.com

	Nginx 설정 분석
	
	Let’s Encrypt에 “이 도메인 인증서 주세요” 요청

	임시 인증 경로 자동 설정

	Let’s Encrypt가 80포트로 접속해 확인

	인증서 발급

	Nginx 설정을 HTTPS로 자동 수정

	Nginx reload

## 발급 후

인증서 위치
ex)
/etc/letsencrypt/live/api.example.com/
├─ fullchain.pem   ← 인증서 (공개)
└─ privkey.pem     ← 개인키 ( 절대 유출 금지)

## certbot이 만들어준 Nginx HTTPS 설정

server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://spring:8080;
    }
}

## HTTP → HTTPS 자동 리다이렉트

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}

사용자가 http://로 와도 자동으로 https://로 이동

301=HTTP 사애코드

## 인증서 자동 갱신

sudo certbot renew --dry-run

	cet’s Encrypt 인증서는 90일짜리 만료 전에 자동 갱신 필수

	certbot이 cron / systemd timer로 하루 2번 자동 체크

	만료 임박 시 자동 갱신 + Nginx reload

## 정리

1. 브라우저가 https://api.example.com 접속
2. Nginx가 인증서 제공
3. TLS Handshake 완료 
4. Nginx가 HTTP로 Spring Boot에 전달
5. Spring 응답
6. 응답을 다시 HTTPS로 암호화해서 브라우저에 전달

server {
    listen 80;
    server_name api.example.com;

    # 모든 HTTP 요청을 HTTPS로 영구 이동
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.example.com;

    # ===== TLS 인증서 설정 =====
    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    # ===== 기본 보안 옵션 =====
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    # ===== Spring Boot로 프록시 =====
    location / {
        proxy_pass http://spring:8080;

        # 원본 요청 정보 전달 
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

proxy_set_header Host $host;:원래 사용자가 요청한 도메인(Host) 을 Spring에게 그대로 전달

proxy_set_header X-Real-IP $remote_addr: 진짜 접속한 사용자의 IP 를 Spring에게 전달.

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for: 클라이언트 IP의 체인(경로) 를 만들어서 전달

proxy_set_header X-Forwarded-Proto $scheme: 사용자가 원래 어떤 프로토콜로 접속했는지
