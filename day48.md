# 무중단 배포

무중단 배포 = “지금 쓰는 서버는 그대로 두고,
새 서버를 미리 준비한 뒤, 스위치만 바꾼다.”

## 왜 배포하면 서비스가 멈출까

서버중지->새 코드 업로드->서버 재시작: 서버가 내려간 시간, 요청을 받을 수 없는 순간이 생긴다(다운타임)->사용자가 에러화면을 보게되고 데이터 꼬이고 장애로 인식된다

## 배포 흐름

| 이름        | 역할       |
| --------- | -------- |
| **Blue**  | 현재 서비스 중 |
| **Green** | 새 버전 준비  |

[사용자] → Nginx → Blue (v1)    서비스 중
                 → Green (v2)   아직 안 씀
                 
1. Green에 새 버전 배포
2. Green 정상 동작 확인(헬스체크)
3. Nginx가 트래픽을 Green으로 전환
4. Blue 종료 (또는 대기)

-> 사용자는 끊김을 못 느낌

## 왜 Nginx가 있어야 가능할까

Nginx가 있으면 앞단에서 어디로 보낼지 결정 및 트래픽 스위치 가능 및 포트/컨테이너 여러개 관리

## 스위치 전환

Nginx 설정 한 줄 바꾸는 것

proxy_pass http://blue:8080;->proxy_pass http://green:8080;으로 바꾸고 reload

## reload vs restart

| 구분     | reload | restart |
| ------ | ------ | ------- |
| 연결 유지  | o      | x       |
| 다운타임   | 없음     | 발생      |
| 무중단 배포 | o      | x      |

## 배포 실패하면 어떻게 할까?(롤백)

green에 문제 생기면 다시 blue로 스위치

ex) Nginx
 ├─ Spring Blue (8081)
 └─ Spring Green (8082)
 
# blue/green 예제

> 브라우저
  ↓  https://api.example.com
Nginx
  ├─ Blue  (Spring v1, 8081)  ← 현재 서비스
  └─ Green (Spring v2, 8082)  ← 새 버전
  
## Spring Boot

Blue → 8081 / Green → 8082

application.yml
ex) server:
  port: ${SERVER_PORT}

## docker-compose
ex) 
version: "3.8"

services:
  blue:
    image: my-dockerhub/ufc-risk-platform:latest
    container_name: spring-blue
    environment:
      - SERVER_PORT=8081
    expose:
      - "8081"

  green:
    image: my-dockerhub/ufc-risk-platform:latest
    container_name: spring-green
    environment:
      - SERVER_PORT=8082
    expose:
      - "8082"

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - blue
      - green
      
## Nginx 설정

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://blue:8081;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

## 실행

docker-compose up -d

브라우저 접속: https://api.example.com

## Green에 새 버전 배포

docker pull your-dockerhub/ufc-risk-platform:new (dockerhub에서 새 이미지 내려받는다)
docker stop spring-green  (현재 실행중인 green커네이너 종료)
docker rm spring-green (멈춘 green 컨테이너 삭제)

docker-compose up -d green

nginc.conf 한줄 변경

proxy_pass http://blue:8081;->proxy_pass http://green:8082;

docker exec nginx nignx -s reload 
	
    기존 연결 유지 및 새 요청만 green으로 간다
    
## 전환 후 상태
브라우저
  ↓
Nginx
  └─ Green (새 버전) 

blue는 이제 대기/백업

## 롤백

문제 발생시 proxy_pass http://blue:8081; docker exec nginx nginx -s reload
