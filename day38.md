# VPC

## VPC란?

VPC = Amazon Web Services 안에 만드는 나만의 가상 네트워크

vpc끼리는 서로 통신 불가

왜 vpc가 필요할까?:

  IP 충돌 방지, 보안 분리, 네트워크 제어 (누가 누구랑 통신?)

## CIDR — “내 네트워크의 크기 정하기”

CIDR = 이 VPC에서 사용할 IP 주소 범위

ex) 10.0.0.0/16  VPC: /16 Subnet: /24

## Subnet

Subnet = VPC 안의 작은 네트워크 조각

반드시 하나의 AZ에 속함

VPC CIDR 범위 안에서 생성

Public / Private는 속성 X
→ 라우팅 설정 결과

## Public vs Private Subnet

인터넷으로 나가는 길이 있느냐 없느냐

### Public Subnet

0.0.0.0/0 → IGW 라우트 테이블

인터넷 접근 가능

웹 서버, API 서버

### Private Subnet

IGW로 가는 길 X

인터넷 직접 접근 불가

DB, 내부 서버

| 계층     | 역할    |
| ------ | ----- |
| VPC    | 완전 격리 |
| Subnet | 영역 분리 |
| AZ     | 장애 분리 |
