# GitHub Actions

저비용으로 CI/CD를 해보자!

Github Actions with

- Heroku

- Netlify   => 이 친구는 정적 사이트용이니 일단 패스

- CodeDeploy + EC2

- Jenkins  => 너무 비싸진 않나...

  

<br /><br />

## 1. Heroku

1. https://start.spring.io/ 에서 다음과 같이 세팅해서 열어준다. (혹은 개인 프로젝트)

   (할때마다 까먹어서 또 적으면 Group는 패키지명이고, Artifact는 빌드명이다.)

   <div align=left>
     <img src='https://user-images.githubusercontent.com/42775225/143015314-bd1251dc-e5b3-4bdb-abb4-7a3f932882a4.png'> </img>
   </div>

   <br /><br />

2. Heroku 공식 홈페이지의 가이드를 따라간다. 

   https://devcenter.heroku.com/articles/deploying-gradle-apps-on-heroku

   -  Procfile 파일 만들기 (확장자 없음)

     내용은

     - [빌드명] 은 위에서 정한 것
     - [version] 은 build.gradle에 version이다.

     ```
     web: java -Dserver.port=$PORT $JAVA_OPTS -jar build/libs/[빌드명]-[version].jar
     ```

   - system.properties 파일 만들기 (필수는 아님)

     https://devcenter.heroku.com/articles/java-support 를 보면, heroku가 자바를 지원하는 방식이다.

     ```
     java.runtime.version=11
     ```

   - gradlew를 사용한다면, /gradle/wrapper 폴더는 gitignore로 빼버리면 안된다.

   <br /><br />



3. heroku 회원가입을 하고 앱을 만들자.

   https://dashboard.heroku.com/apps 에서 new로 만들 수 있다.

   <br /><br />

   

4. 이제 레포지토리에 Actions탭 > 'set up a workflow yourself'를 선택하자.

   (이미 만든적 있다면 New workflow)

   ![image](https://user-images.githubusercontent.com/42775225/143018586-abc0994f-55bc-44cf-96e3-811105608a3d.png)

   그러면 루트에 .github/workflows 라는 폴더가 생겼을 것이다. 거기에 있는 yml파일의 내용에 heroku 값들을 넣어주자.

   ```yaml
   name: [원하는 이름]
   
   on:
     push:
       branches: [ main ]
   
   jobs:
     build:
       runs-on: ubuntu-latest    # 돌릴 OS
       
       steps:
         - uses: actions/checkout@v2
         
   #      - name: Build project    # 로컬에서 build해서 확인하고 배포할 것이기 때문에 이건 굳이 필요는 없음 
   #        run: ./gradlew build
     
         - name: Deploy to Heroku
           uses: AkhileshNS/heroku-deploy@v3.12.12
           with:
             heroku_api_key: ${{ secrets.heroku_api_key }}
             heroku_email: ${{ secrets.heroku_email }}
             heroku_app_name: ${{ secrets.heroku_app_name }}
   ```

   - heroku_api_key
   - heroku_email
   - heroku_app_name

   값만 넣으면 되는데, 이거는 Settings탭 > Secrets에서 값을 숨겨서 변수로 사용하자.

   (api_key는 Account Settings의 맨 밑에 있다.)



<br /><br />



## 2. CodeDeploy + EC2

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





<br /><br />



## 대체제) Jenkins

https://github.com/jenkinsci/jenkins

https://jojoldu.tistory.com/139



#### 서버 설치

1. Jenkins 서버 설치

   https://hub.docker.com/_/jenkins 에 들어간다. (Docker)



#### 로컬 설치

1. Jenkins 로컬 호스트 설치

   https://www.jenkins.io/download/ 에서 각자 OS에 맞는거 설치한다.

   ```shell
   brew install jenkins
   ```

2. 









