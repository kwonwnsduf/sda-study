# 로그
로그(Log) = 서버가 남긴 기록

## 로그 종류

### Access Log(접근 로그)

“누가, 어떤 요청을 했는지”

ex) 127.0.0.1 - - [12/Mar/2026:10:21:32] "GET /api/users HTTP/1.1" 200

포함 정보: 클라이언트 IP, 요청 URL, HTTP 메소드, 응답 코드

### Error Log

“서버 내부에서 문제가 생겼을 때”

ex) [error] connect() failed (111: Connection refused)

### Application Log

Spring Boot가 직접 남기는 로그

ex) INFO  UserService - userId=10 로그인 시도
ERROR PaymentService - 결제 실패 (잔액 부족)

브라우저
  ↓
Nginx  → access.log / error.log
  ↓
Spring Boot → application.log
  ↓
Docker → container logs

1. Nginx access log

요청이 서버까지 왔는지

2. Nginx error log

프록시/연결 문제

3. Spring Boot 로그

로직/예외/DB 문제

4. DB 로그

# CloudWatch 로그

CloudWatch는 “서버 밖에 있는 중앙 로그 관제실”이다.
서버가 죽어도 로그는 남아 있어야 한다.

CloudWatch 없으면

	EC2 접속(SSH) 못 하면 → 로그 못 봄

	서버 죽으면 → 로그 같이 사라질 수 있음

	여러 서버면 → 하나하나 접속해서 확인

CloudWatch 있으면

	AWS 콘솔에서 언제든 로그 확인

	서버 재시작/장애에도 로그 보존

	여러 서버 로그를 한 화면에서 검색

Spring Boot / Nginx / Docker
        ↓
   CloudWatch Agent
        ↓
   AWS CloudWatch Logs
        ↓
  브라우저 (AWS 콘솔)
  
## CloudWatch Logs 개념

### Log Group

로그 묶음 (프로젝트 단위)

### Log Stream

개별 로그 줄기 (서버/컨테이너 단위)

### Log Event

실제 로그 한 줄

## 로그수집

ex) | 대상          | 로그                 |
| ----------- | ------------------ |
| EC2         | 시스템 로그             |
| Docker      | 컨테이너 stdout/stderr |
| Spring Boot | application log    |
| Nginx       | access / error log |

### EC2에 CloudWatch Agent 설치

sudo apt update
sudo apt install amazon-cloudwatch-agent -y

### 로그 수집 설정 파일 작성

sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json

{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/lib/docker/containers/*/*.log",
            "log_group_name": "/ufc-risk-platform/docker",
            "log_stream_name": "{instance_id}",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}

"logs": 로그 수집 관련 섹션 시작
"logs_collected" : 어떤 방식으로 로그를 모을건지
"files" : 파일기반 수집
"collect_list" : 수집할 로그 파일 규칙을 여러 개 넣을 수 있는 리스트
"file_path": "/var/lib/docker/containers/*/*.log" -> docker컨테이너 로그 파일들이 저장되는 위치 지정
"log_group_name": CloudWatch Logs의 Log Group 이름.
"log_Stream_name": Log Group 안에서 “줄기(스트림)” 이름을 어떻게 지을지.


Docker 컨테이너 로그 전부 수집

CloudWatch에 /ufc-risk-platform/docker로 전송

서버별로 stream 분리

### Agent 실행

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
-s

sudo: root 권한 필요
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl: cloutwatch agent제어 프로그램
-a fetch-config: 설정파일을 가져와서 적용한다
-m ec2: 이 agent는 EC2환경에서 실행 중이다
-c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json: 이 파일에 있는 설정을 사용해라
-s: 설정 적용한 뒤, agent를 바로 실행(start)해라


### CloutWatch 콘솔에서 확인

Docker 컨테이너는 죽으면 재시작되면 로그 파일 위치가 헷갈릴 수 있다.

CloutWatch에 있으면 컨테이너 재식작x 문제 /과거 로그도 검색 가능
