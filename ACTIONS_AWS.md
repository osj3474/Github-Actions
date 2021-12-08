# Actions + CodeDeploy + EC2

우선 EC2 에 배포 테스트부터 확인할 필요가 있다.

요금은

- 온디맨드

- 아시아(서울)
- Linux

을 기준으로 다음과 같다.



![image](https://user-images.githubusercontent.com/42775225/144217850-b1aeac3d-232c-4cfe-b94b-f1bc2a44a1da.png)

https://aws.amazon.com/ko/ec2/pricing/on-demand/



최소 사양을 t3.micro로 잡으면 한 달에 11232원 정도 (환율 : 1200원 기준)가 예상된다.

EC2에는 jar 배포 파일과 MySQL DB서버만 있을 예정이며,

현재 develop브랜치 기준으로 모든 API 에 대하여 tps 1을 때리고 모니터링 예정이다.



<br />

#### 1) 로컬

우선 jar를 만든다

```
./gradlew build
```

그리고 AWS EC2로 올린다.

```
scp -i [키이름].pem dev-event-server-0.0.1-SNAPSHOT.jar ubuntu@[서버IP]:~/
```



<br />

#### 2) 서버

jvm부터 설치한다.

https://davelogs.tistory.com/71

```
$ sudo apt-get update
$ sudo apt-get upgrade

# JAVA11 설치
$ sudo apt-get install openjdk-11-jdk
```



mysql 설치한다.

https://velog.io/@seungsang00/Ubuntu-%EC%9A%B0%EB%B6%84%ED%88%AC%EC%97%90-MySQL-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0

```
sudo apt-get update # 우분투 서버 업데이트
sudo apt-get install mysql-server # mysql-server 설치
sudo ufw allow mysql # 외부 접속 기능 설정 (포트 3306 오픈)
sudo systemctl start mysql # MySQL 실행
sudo systemctl enable mysql # Ubuntu 서버 재시작시 MySQL 자동 재시작
sudo /usr/bin/mysql -u root -p # MySQL 접속

CREATE DATABASE dev_event default CHARACTER SET UTF8; # DB생성
```

참고로 현재 port 9000으로 픽스 해두었으므로, ec2에서 방화벽(인바운드 규칙) 9000 열어야 된다.



<br />

#### 3) 모니터링

http://[서버IP]:9000/admin/v1/events/2021/7 로 

해보면 기본적인 API 응답 가능한 것 확인되었고,

이제 postman으로 모니터링 설정을 준비한다.







**\< 사용법 >**

<a href="https://isntyet.github.io/deploy/github-action%EA%B3%BC-aws-code-deploy%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-spring-boot-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0(1)/" target="_blank">JaeYoung 님 블로그</a> 를 참고하여 작성하였습니다.



1. EC2에 ruby설치 후 code-deploy agent를 설치

   ```shell
   sudo apt-get update
   sudo apt-get install ruby
   sudo apt-get install wget
   cd /home/ubuntu
   
   wget https://`bucket-name`.s3.`region-identifier`.amazonaws.com/latest/install
   chmod +x ./install
   sudo ./install auto
   ```

   bucket-name과 region-identifier은 

   <a href="https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names" target="_blank">AWS 공식 문서</a>를 기준으로 설정하되 저는 다음과 같습니다.

   ```shell
   wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install
   ```

   (참고로, 현재 EC2에는 jvm과 mysql이 설치되어 있어서 자바 어플리케이션이 실행가능한 상태에서 시작하였습니다.)



2. AWS 세팅
   - EC2 IAM 설정
   - CodeDeploy 어플리케이션 생성 및 IAM 설정
   - S3 생성
   - 사용자 생성 (accessKey, secretKey 필요) 





3. EC2 배포스크립트 작성

   프로젝트 루트 경로에

   - appspec.yml

     ```yaml
     version: 0.0
     os: linux
     
     files:
       - source: /
         destination: /home/ubuntu/sangjin-deploy
     permissions:
       - object: /home/ubuntu/sangjin-deploy/
         owner: ubuntu
         group: ubuntu
     hooks:
       AfterInstall:
         - location: scripts/deploy.sh
           timeout: 60
           runas: ubuntu
     
     ```

   - deploy.sh

     ```shell
     #!/usr/bin/env bash
     
     REPOSITORY=/home/ubuntu/sangjin-deploy
     cd $REPOSITORY
     
     APP_NAME=actions
     JAR_NAME=$(ls $REPOSITORY/build/libs/ | grep 'SNAPSHOT.jar' | tail -n 1)
     JAR_PATH=$REPOSITORY/build/libs/$JAR_NAME
     
     CURRENT_PID=$(pgrep -f $APP_NAME)
     
     if [ -z $CURRENT_PID ]
     then
       echo "> 종료할것 없음."
     else
       echo "> kill -9 $CURRENT_PID"
       kill -15 $CURRENT_PID
       sleep 5
     fi
     
     echo "> $JAR_PATH 배포"
     nohup java -jar $JAR_PATH > /dev/null 2> /dev/null < /dev/null &
     
     
     ```



4. .github/workflows에서 yml 생성

   ```yaml
   # This is a basic workflow to help you get started with Actions
   
   name: github actions + heroku
   
   on:
     push:
       branches: [ main ]
   
     # Allows you to run this workflow manually from the Actions tab
     workflow_dispatch:
   
   env:
     PROJECT_NAME: CDapp
   
   # A workflow run is made up of one or more jobs that can run sequentially or in parallel
   jobs:
     # This workflow contains a single job called "build"
     build:
       # The type of runner that the job will run on
       runs-on: ubuntu-latest
   
       # Steps represent a sequence of tasks that will be executed as part of the job
       steps:
         # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
         - uses: actions/checkout@v2
   
         - name: Make zip file
           run: zip -qq -r ./$GITHUB_SHA.zip .   # GITHUB_SHA는 예약변수로, 커밋 해시값
           shell: bash
   
         - name: Configure AWS credentials
           uses: aws-actions/configure-aws-credentials@v1
           with:
             aws-access-key-id: ${{ secrets.accessKey }}
             aws-secret-access-key: ${{ secrets.secretKey }}
             aws-region: ${{ secrets.region }}
   
         - name: Upload to S3
           run: aws s3 cp --region ap-northeast-2 ./$GITHUB_SHA.zip s3://sangjin-deploy/$PROJECT_NAME/$GITHUB_SHA.zip
   
         - name: Code Deploy
           run: aws deploy create-deployment --application-name CDapp --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name CDapp --s3-location bucket=sangjin-deploy,bundleType=zip,key=$PROJECT_NAME/$GITHUB_SHA.zip
   ```

   

5. 성공 ⭐️

   <br />

   ![image](https://user-images.githubusercontent.com/42775225/144769049-09ba47a0-abf0-4e57-b477-f69ec0654b22.png)



