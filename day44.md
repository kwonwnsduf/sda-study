# AWS 아키텍처

> 사용자
  ↓
Application Load Balancer
  ↓
EC2 (AZ-A)   EC2 (AZ-B)
   │            │
   └───→ RDS (Multi-AZ)
   │
   └───→ S3 (파일)
   
## VPC/Subnet

VPC: 나만의 네트워크 공간

Public Subnet: ALB, NAT

Private Subnet: EC2, RDS

## Application Load Balancer

사용자 요청 분산

살아있는 EC2만 선택

서버 교체/장애를 사용자에게 숨김

## EC2 (다중 AZ)

실제 애플리케이션 실행

AZ-A / AZ-B 분산 배치

## RDS (Multi-AZ)

데이터 저장

Primary + Standby

장애 시 자동 전환

->DB 고가용성 확보

## S3

이미지 / 파일 / 백업 저장

서버와 분리된 스토리지

->서버 무상태 설계

## IAM

IAM User: 사람 로그인

IAM Role: EC2가 S3/RDS 접근
