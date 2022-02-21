### 8.2 배포 스크립트 만들기

배포

- 작성한 코드를 실제 서버에 반영하는 것
1. git clone 혹은 git pull을 통해 새 버전의 프로젝트 받음
2. Gradle이나 Maven을 통해 프로젝트 테스트와 빌드
3. EC2 서버에서 해당 프로젝트 실행 및 재실행

쉘 스크립트

- .sh라는 파일 확장자를 가진 파일
- 리눅스에서 기본적으로 사용할 수 있는 스크립트 파일의 한 종류

빔

- 리눅스 환경과 같이 GUI가 아닌 환경에서 사용할 수 있는 편집 도구

```bash
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=springboot2-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=${pgrep -f ${PROJECT_NAME}.*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &

```

- REPOSITORY=/home/ec2-user/app/step1
1. 프로젝트 디렉토리 주소는 스크립트 내에서 자주 사용하는 값이기 때문에 이를 변수로 저장
2. 쉘에서는 타입 없이 선언하여 저장
3. 쉘에서는 $ 변수명으로 변수를 사용할 수 있음

- cd $REPOSITORY/$PROJECT_NAME
1. 제일 처음 git clone 받았던 디렉토리로 이동

- git pull
1. 디렉토리 이동 후, master 브랜치의 최신 내용을 받음

- ./gradlew build
1. 프로젝트 내부의 gradlew로 build를 수행함

- cp ./build/libs/*.jar $REPOSITORY/
1. build의 결과물인 jar 파일을 복사해 jar 파일을 모아둔 위치로 복사

- CURRENT_PID=$(pgrep -f springboot-webservice)
1. 기존에 수행 중이던 스프링부트 애플리케이션을 종료
2. pgrep은 process id만 추출하는 명령어
3. -f 옵션은 프로세스 이름으로 찾음

- if ~ else ~ fi
1. 현재 구동 중인 프로세스가 있는지 없는지를 판단해서 기능 수행
2. process id 값을 보고 프로세스가 있으면 해당 프로세스를 종료

- JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)
1. 새로 실행할 jar 파일명을 찾음
2. 여러 jar 파일이 생기기 때문에 tail -n로 가장 나중에 jar 파일을 변수에 저장

- nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
1. 찾은 jar 파일명으로 해당 jar 파일을 nohup으로 실행
2. 스프링 부트의 장점으로 특별히 외장 톰캣을 설치할 필요가 없음
3. 내장 톰캣을 사용해서 jar 파일만 있으면 바로 웹 어플리케이션 서버를 실행할 수 있음
4. 자바를 실행할 때는 java -jar 명령어를 사용, 이렇게 하면 사용자가 터미널 접속을 끊을 때 애플리케이션도 같이 종료
5. 애플리케이션 실행자가 터미널을 종료해도 애플리케이션은 계속 구동될 수 있도록 nohup 명령어 사용

### 8.3 외부 Security 파일 등록하기

-Dspring.config.location

1. 스프링 설정 파일 위치를 지정
2. 기본 옵션들을 담고 있는 application.properties과 OAuth 설정들을 담고 있는 application-oauth.properties의 위치를 지정
3. classpath가 붙으면 jar 안에 있는 resources 디렉토리를 기준으로 경로가 생성됨
4. application-oauth.properties는 절대 경로를 사용

⇒ 현재 방식의 문제점

- 수동 실행되는 Test
1. 본인이 짠 코드가 다른 개발자의 코드에 영향을 끼치지 않는지 확인하기 위해 전체 테스트를 수행해야 함
2. 현재 상태에서는 항상 개발자가 작업을 진행할 때마다 수동으로 전체 테스트를 수행해야 함

- 수동 Build
1. 다른 사람이 작성한 브랜치와 본인이 작성한 브랜치가 합쳐졌을 때 이상이 없는지는 Build를 수행해야만 알 수 있음
2. 이를 매번 개발자가 직접 실행해야함
