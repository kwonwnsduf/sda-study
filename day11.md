## EC2 사전준비
### EC2에 프로젝트 클론
> ssh -i C:\keys\sda-key.pem ec2-user@퍼블릭IP

접속명령

> git --version
sudo yum update -y
sudo yum install git -y

EC2에 git 설치 확인/설치

> cd ~
git clone https://github.com/yourname/sda-project.git
cd sda-project
ls

### Docker/Docker Compose 설치
> docker -v
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
docker compose version
sudo curl -L \
"https://github.com/docker/compose/releases/download/v2.25.0/docker-compose-linux-x86_64" \
-o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose -v

EC2에서 Docker설치/docker 데몬실행+부팅 시 자동 실행->ec2-user가 sudo 없이 docker쓰게하기->확인->Docker Compose 설치/확인

### EC2에서 수동 배포 1회 성공 확인
> docker compose up -d --build

## GitHub Secrets 설정
GitHub 저장소 → Settings →Secrets → Actions->New Repository secret

| Key        | Value                        |
| ---------- | ---------------------------- |
| `EC2_HOST` | 퍼블릭 IP                       |
| `EC2_USER` | ec2-user                     |
| `EC2_KEY`  | EC2 pem 키 내용                 |
| `EC2_PATH` | `/home/ec2-user/sda-project` |

EC2_KEY확인
> cat my-key.pem

## CD 포함된 GitHub Actions 예제
> name: CI-CD
on:
  push:
    branches: [ "main" ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
	 - name: SSH to EC2 & Deploy
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            cd ${{ secrets.EC2_PATH }}
            git pull
            docker compose down
            docker compose up -d --build
          
>  .github/
 └─ workflows/
     └─ ci-cd.yml
 
프로젝트 루트에서
> mkdir -p .github/workflows

파일 생성
> touch .github/workflows/ci-cd.yml


appleboy/ssh-action은 “원격 서버에 SSH로 접속해서 명령어 실행”을 해주는 액션
이 액션이 GitHub 러너에서 EC2로 접속한다.

with는 이 액션에 전달하는 설정값.

Script:EC2에 접속한 뒤 실제로 실행할 명령어들.

git pull: EC2에 이미 clone되어 있는 저장소를 최신 커밋으로 업데이트

워크플로우 파일 push해서 실행 시키기
> git add .github/workflows/ci-cd.yml
git commit -m "Add CD workflow"
git push origin main
