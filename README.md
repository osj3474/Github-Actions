# GitHub Actions

저비용으로 CI/CD를 해보자!

Github Actions with

- Heroku
- Netlify
- 

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





