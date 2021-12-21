# Heroku + GCP

<br />

### 3-1. GCP Compute Engine + MySQL

```
STEP 1. ec2-micro 인스턴스에 MySQL 설치
        설치 커멘드 기록

STEP 2. ec2-micro 인스턴스 비용 정책 기록
```

<br />

> STEP 1.

1. 설치 스크립트 생성

   ***mysql.sh***

   ```shell
   sudo apt-get -y update # 우분투 서버 업데이트
   sudo apt-get -y install mysql-server # mysql-server 설치 (혹은 mariadb-server)
   sudo ufw allow mysql # 외부 접속 기능 설정 (포트 3306 오픈)
   sudo systemctl start mysql # MySQL 실행
   sudo systemctl enable mysql # Ubuntu 서버 재시작시 MySQL 자동 재시작
   
   
   query="CREATE DATABASE [DB명] default CHARACTER SET UTF8; # DB생성
   
   CREATE USER '[사용자명]'@'[Spring서버 IP]' IDENTIFIED BY '[비밀번호]'; # 사용자 생성
   FLUSH PRIVILEGES;
   
   GRANT ALL PRIVILEGES ON [DB명].* TO '[사용자명]'@'[Spring서버 IP]';
   FLUSH PRIVILEGES;
   SHOW GRANTS FOR'[사용자명]'@'[Spring서버 IP]';"
   
   sudo /usr/bin/mysql -u root -D mysql -e "${query}"  # DB 쿼리 실행
   ```

2. 스크립트 실행

   ```shell
   ./mysql.sh
   ```

<br />



> STEP 2.

<a href="https://cloud.google.com/free/docs/gcp-free-tier?hl=ko&_ga=2.142668413.-979674017.1638945935#free-tier-usage-limits" target="_blank">GCP 무료 프로그램</a> 을 참고하면 다음과 같은 안내글을 볼 수 있습니다.



![image](https://user-images.githubusercontent.com/42775225/145178387-676668c5-7c7d-409e-bca5-6ff699d670d5.png)

- 특정 리전 (오리건: `us-west1`, 아이오와: `us-central1`, 사우스캐롤라이나: `us-east1`)
- 스토리지 30GB

까지는 월 720시간까지 무료로 사용할 수 있습니다.

<br />

그래서 다음과 같이 설정하였습니다.

![image](https://user-images.githubusercontent.com/42775225/145178421-0dbd7bb6-57e3-411c-9e4f-148548641e68.png)



(월별 예상 가격은 720시간 초과 후의 가격이지 않을까 생각됩니다.)



<br />
<br />

### 3-2. 헤로쿠 연동 테스트

```text
STEP 1.  GCP 인스턴스에 설치한 MySQL에 헤로쿠 연동
          MySQL 정보는application-real.yml 설정정보 추가

STEP 2. DEV GROUP real data로 insert

STEP 3. API 몇번 호출해보고 평균 응답 ms 기록
```

<br />

> STEP 1.

1. GCP MySQL 서버 IP 확인 후 프로젝트 설정 값 변경

   - MySQL IP : xx.xxx.xx.xxx

   - appliction-real.yml

     ```yaml
     spring:
       datasource:
         driver-class-name: com.mysql.cj.jdbc.Driver
         url: jdbc:mysql://xx.xxx.xx.xxx:3306/dev_event?characterEncoding=UTF-8&serverTimezone=UTC
         username: [사용자명]
         password: [비밀번호]
     ```

   ⛱ 주의할 점

   - mysql 서버의 사용자를 만들때 `스프링 서버 ip` 만 허용해서 만들것
   - db서버 ip오픈은 `스프링 서버 ip` 만 허용할 것



2. MySQL 서버 오픈

   ```shell
   vi /etc/mysql/mariadb.conf.d/50-server.cnf
   ```

   ***50-server.cnf***

   ```
   bind-address = [스프링 서버 ip]
   ```

   ```shell
   service mysql restart
   netstat -lntp  # 3306번 포트 열린 ip 확인  
   ```

   

3. 이렇게 하면 배포는 완료~!

   ![image](https://user-images.githubusercontent.com/42775225/145210435-d7a2bfec-24b0-4dd5-8810-8cb04653e2fc.png)





cf) heroku 참고

```shell
$ heroku login

$ heroku git:remote -a sangjin-test

$ git add .
$ git commit -am "make it better"
$ git push heroku master
```

