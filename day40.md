## 전체 구조

인터넷
 ↓
[NACL]  ← Subnet 단
 ↓
[Security Group] ← EC2 단
 ↓
EC2

## Security Group (SG)

Security Group = 인스턴스 단위의 상태 저장(Stateful) 방화벽

EC2 바로 앞에서 지킴 “이 서버에 들어와도 돼?” “이 서버에서 나가도 돼?”

인스턴스 단위

	EC2에 직접 붙음

	같은 Subnet이어도 EC2마다 다를 수 있음
    
상태 저장 (Stateful)
	
    들어오는 것(Inbound)만 허용하면 응답 나가는 건 자동 허용

ex) Inbound: HTTP 80 허용 Outbound: 아무 설정 안 해도
    → 웹 응답 정상

허용 규칙만 있음

	DENY 규칙 X

	기본은 전부 차단

	허용한 것만 통과
 
 Inbound
- TCP 80  | 0.0.0.0/0
- TCP 22  | 내 IP

Outbound
- ALL     | 0.0.0.0/0

## NACL

NACL = 서브넷 단위의 상태 비저장(Stateless) 방화벽

Subnet 입구에서 전부 검사

이 Subnet으로 들어오거나 나가는
-> 모든 트래픽에 적용

Subnet 단위

	Subnet에 속한 모든 EC2에 강제 적용

	EC2가 선택 불가

상태 비저장 (Stateless)

	들어오는 것 허용해도 나가는 것도 반드시 허용 규칙 필요

허용 + 거부 가능

	ALLOW

	DENY

	우선순위 번호 있음

Inbound
100 ALLOW TCP 80  0.0.0.0/0
110 ALLOW TCP 22  내 IP
*
DENY ALL

Outbound
100 ALLOW TCP 1024-65535 0.0.0.0/0
*
DENY ALL

->응답 포트(1024~65535) 허용 필수

## SG vs NACL 비교
 
 | 구분    | Security Group | NACL      |
| ----- | -------------- | --------- |
| 적용 단위 | EC2            | Subnet    |
| 상태    | Stateful       | Stateless |
| 규칙    | 허용만            | 허용 + 거부   |
| 응답 처리 | 자동 허용          | 명시 필요     |
| 난이도   | 쉬움             | 어려움       |

## 접속 실패 시 사고 흐름

> 1. Route Table (길 있음?)
2. NACL (Subnet 차단?)
3. Security Group (포트 열림?)
