# 네트워크 흐름(IGW NAT Route Table)

## Route Table

Route Table = 목적지별로 어디로 보내야 하는지 적힌 길 안내판

| Destination | Target |
| ----------- | ------ |
| 10.0.0.0/16 | local  |
| 0.0.0.0/0   | ???    |

10.0.0.0/16 → local
 VPC 내부 통신

0.0.0.0/0 → ?
 외부로 나갈 때 어디로 갈지
 
## IGW (Internet Gateway)

###  public subnet 흐름

IGW = VPC가 인터넷과 직접 연결되는 출입문

1. EC2 (10.0.1.10) → 8.8.8.8

2. Route Table 확인
 
 0.0.0.0/0 → IGW

3. IGW 통과

4. 공인 IP로 변환

5. 인터넷 도착

->나감/들어옴 O

### IGW 특징

Public Subnet만 사용

외부 → 내부 양방향 가능

보안은 Security Group / NACL이 담당

## NAT Gateway

NAT Gateway = Private Subnet이 ‘밖으로만’ 나가게 해주는 중계소

### 왜 NAT가 필요할까? 

Private Subnet 서버도: OS 업데이트, 라이브러리 다운로드, 외부 API 호출은 해야 함.

 그런데 IGW를 직접 붙이면?
-> Private 의미가 사라짐

->그래서 NAT 등장

### private subnet 흐름

1. Private EC2 (10.0.2.10) → google.com

2. Route Table 확인
	0.0.0.0/0 → NAT Gateway

3. NAT Gateway (Public Subnet에 위치)

4. IGW

5. 인터넷

 외부 → Private 직접 접근 불가
 
### NAT의 핵심 성질

Outbound only

Inbound X

NAT 자체는 Public Subnet에 있음

## Public vs Private 네트워크 흐름 비교 

| 구분     | Public Subnet | Private Subnet |
| ------ | ------------- | -------------- |
| 외부 접속  | O           | X             |
| 외부로 나감 | O           | O (NAT)        |
| Route  | IGW           | NAT            |
| 용도     | Web / ALB     | DB / 내부 서버     |

## Route Table 예시

### Public Subnet Route Table

10.0.0.0/16   → local
0.0.0.0/0     → IGW

### Private Subnet Route Table

10.0.0.0/16   → local
0.0.0.0/0     → NAT Gateway

## 왜 이렇게 나눴을까

전부 IGW에 붙이면 DB까지 외부 노출 및 침입 시 전체 털림

*인터넷은 꼭 필요한 곳만/내부 자원은 절대 직접 노출 X/ 나가는 것과 들어오는 것을 분리
