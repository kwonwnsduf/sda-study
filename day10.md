## CI/CD는 왜 필요할까?
그 전까지 흐름 복기
> [로컬]
Spring Boot 코드 작성
↓
./gradlew bootJar
↓
docker build
↓
docker compose up

### 문제점

1.매번 직접 서버 접속

2.실수 가능성 큼

3.팀 프로젝트에서는 거의 불가능

### 필요 이유

1. 배포 자동화

2. 휴먼 에러 감소

## CI / CD 개념 
### CI
코드가 합쳐질 때 자동으로 검사

push / PR 발생

자동 테스트

자동 빌드
> git push → 자동으로 gradle build 실행

### CD

빌드된 결과를 자동으로 서버에 배포

Docker 이미지 생성

서버에 반영

컨테이너 재시작

> build 성공 → docker image 생성 → EC2에서 실행

## 전체구조
> [개발자]
git push
   ↓
[GitHub Actions]
- gradle build
- docker build
- docker image 생성
   ↓
[Docker Hub or Registry]
이미지 저장
   ↓
[EC2 서버]
docker pull
docker compose up -d

## GitHub Actions란?

GitHub에 내장된 CI/CD 자동화 도구

별도 서버 필요 X

.github/workflows/*.yml 만 있으면 됨

## 프로젝트 구조
> .github/
 └─ workflows/
     └─ ci.yml
Dockerfile
docker-compose.yml

## 기본 CI Workflow 예제
> name: CI
on:
  push:
    branches: [ "main" ]
jobs:
  build:
    runs-on: ubuntu-latest
  steps:
      - name: Checkout code
        uses: actions/checkout@v4
	  - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
		- name: Build with Gradle
        run: ./gradlew clean build

### on
“이 자동 작업을 언제 실행할까?”를 정하는 구역.

### push
누군가가 이 저장소에 push 하면 실행하겠다는 뜻.

### branches: ["main"]
push가 아무 브랜치나가 아니라, main 브랜치에 들어갈 때만 실행

### jobs
워크플로우 안에서 실행되는 작업 묶음(Job)들을 정의하는 구역

### build:
job 이름이 build라는 뜻

### runs-on: ubuntu-latest
이 job을 실행할 “가상 컴퓨터”의 OS를 지정.

ubuntu-latest = GitHub가 제공하는 최신 Ubuntu 실행 환경

### steps:
build job 안에서 순서대로 실행되는 단계들.

### Checkout code
name: Checkout code: 이 단계 이름

uses: actions/checkout@v4:누가 만들어 둔 액션(기능 묶음)을 가져다 쓰겠다”

 actions/checkout은 GitHub 공식 액션이고,

 이 저장소 코드를 Ubuntu 머신에 git clone(체크아웃) 해준다

### JDK 설치

uses: actions/setup-java@v4

 Java를 설치/설정해주는 공식 액션.

 “Ubuntu에 자바를 깔아주고, JAVA_HOME 같은 환경변수도 잡아줌.”

with: (액션에 옵션 전달)

 setup-java에게 “자바를 어떻게 설치할지” 알려주는 설정값.

java-version: '17'

 자바 17 버전 설치

distribution: 'temurin'

자바 배포판(벤더) 선택.

### gradle빌드 실행

run:

“Ubuntu 터미널에서 이 명령어를 그대로 실행해라”는 뜻.

./gradlew

프로젝트에 들어있는 Gradle Wrapper 실행 파일.

## Docker까지 연결하면
> - name: Build Docker Image
  run: docker build -t myapp .
- name: Run Container
  run: docker run -d -p 8080:8080 myapp
