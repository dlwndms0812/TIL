### 9.1 CI & CD 소개

- CI(Continuous Integration-지속적 통합)
1. 코드 버전 관리를 하는 VCS 시스템(Git, SVN 등)에 PUSH가 되면 자동으로 테스트와 빌드가 수행되어 안정적인 배포 파일을 만드는 과정
2. 규칙
    - 모든 소스 코드가 살아 있고(현재 실행되고) 누구든 현재의 소스에 접근할 수 있든 단일 지점을 유지할 것
    - 빌드 프로세스를 자동화해서 누구든 소스로부터 시스템을 빌드하는 단일 명령어를 사용할 수 있게 할 것
    - 테스팅을 자동화해서 단일 명령어로 언제든지 시스템에 대한 건전한 테스트 수트를 실행할 수 있게 할 것
    - 누구나 현재 실행 파일을 얻으면 지금까지 가장 완전한 실행 파일을 얻었다는 확신을 하게 할 것

- CD(Continuous Deployment-지속적인 배포)
1. 빌드 결과를 자동으로 운영 서버에 무중단 배포까지 진행되는 과정

⇒ 푸시가 될때마다 코드를 병합하고, 테스트 코드와 빌드를 수행하며 자동으로 코드 통합, 배포 자동화로 인해 개발자들이 개발에만 집중 가능

### 9.2 Travis CI 연동하기

```yaml
language:java
jdk:
  -openjdk8

branches:
  only:
    -master

#Travis CI 서버의 home
cache:
  directories:
    -'$HOME/.m2/repository'
    -'$HOME/.gradle'
script: "./gradles clean build"

#CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      -dlwndms0812@naver.com
```

- branches
1. Travis CI를 어느 브랜치가 푸시될 때 수행할 지 지정
2. 현재 옵션은 오직 master 브랜치에 push될 때만 수행

- cache
1. 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여, 같은 의존성은 다음 배포 때부터 다시 받지 않도록 설정

- script
1. master 브랜치에 푸시되었을 때 수행하는 명령어
2. 여기서는 프로젝트 내부에 둔 gradlew을 통해 clean&build를 수행

- notifications
1. Travis CI 실행시 자동으로 알람이 가도록 설정

### 9.3 Travis CI와 AWS S3 연동하기

- AWS S3
1. 파일들을 저장하고 접근 권한을 관리, 겁색 등을 지원하는 파일 서버의 역할을 함
2. 게시글을 쓸 때 나오는 첨부파일 등록을 구현할 때 많이 이용
3. 버킷 보안과 권한 설정시 모든 퍼블릭 액세스를 차단 해야함

1. jar 전달: Travis CI → AWS S3
2. 배포 요청: Travis CI → AWS CodeDeploy
3. jar 전달: AWS S3 → AWS CodeDeploy
4. 배포: AWS CodeDeploy → AWS EC2

⇒ CodeDeploy는 저장 기능이 없기에 Travis CI가 빌드한 결과물을 S3이 받아서 CodeDeploy가 가져갈 수 있게 함

```yaml
language:java
jdk:
  -openjdk8

branches:
  only:
    -master

#Travis CI 서버의 home
cache:
  directories:
    -'$HOME/.m2/repository'
    -'$HOME/.gradle'
script: "./gradles clean build"

before_deploy:
  -zip -r springboot2-webservice *
  -mkdir -p deploy
  -mv springboot2-webservice.zip deploy/springboot2-webservice.zip

deploy:
  -provider: s3
   access_key_id: $AWS_ACCESS_KEY #Travis repo settings에 설정된 값
   secret_access_key: $AWS_SECRET_KEY #Travis repo settings에 설정된 값
   bucket: springboot-build #S3 버킷
   region: ap-northeast-2
   skip_cleanup: true
   acl: private #zip 파일을 private로
   local_dir: deploy #before_deploy에서 생성한 디렉토리
   wait-until-deployed: true

#CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      -dlwndms0812@naver.com
```

- before_deploy
1. deploy 명령어가 실행되기 전에 수행
2. CodeDeploy는 Jar 파일은 인식하지 못하므로 Jar+기타 설정 파일들을 모아 압축함

- zip -r springboot2-webservice
1. 현재 위치의 모든 파일을 springboot2-webservice 이름으로 압축
2. 명령어의 마지막 위치는 본인의 프로젝트 이름이어야 함

- mkdir -p deploy
1. deploy라는 디렉토리를 Travis CI가 실행 중인 위치에서 생성

- mv springboot2-webservice.zip deploy/springboot2-webservice.zip
1. springboot2-webservice.zip 파일을 deploy/springboot2-webservice.zip으로 이동시킴

- deploy
1. S3로 파일 업로드 혹은 CodeDeploy로 배포 등 외부 서비스와 연동될 행위들을 선언

- local_dir: deploy
1. 앞에서 생성한 deploy 디렉토리를 지정
2. 해당 위치의 파일들만 S3로 전송

### 9.4 Travis CI와 AWS S3, CodeDeploy 연동하기

- IAM의 역할
1. AWS 서비스에만 할당할 수 있는 권한
2. EC2, CodeDeploy, SQS 등

- IAM의 사용자
1. AWS 서비스 외에 사용할 수 있는 권한
2. 로컬 PC, IDC 서버 등

**배포 삼형제**

- Code Commit
1. 깃허브와 같은 코드 저장소의 역할
2. 프라이빗 기능을 지원한다는 강점이 있지만, 현재 깃허브에서 무료로 프라이빗 지원을 하고 있어서 거의 사용하지 않음

- Code Build
1. Travic CI와 마찬가지로 빌드용 서비스
2. 멀티 모듈을 배포해야 하는 경우 사용해 볼만하지만, 규모가 있는 서비스에서는 대부분 젠킨스/팀시티 등을 이용하여 거의 사용하지 않음

- CodeDeploy
1. AWS의 배포 서비스
2. 앞에서 언급한 다른 서비스들은 대체제가 있고, 딱히 대체제보다 나은 점이 없지만 CodeDeploy는 대체제가 없음
3. 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포 등 많은 기능을 지원

**Travis CI, S2, CodeDeploy 연동**

```yaml
version: 0.0
os: linux
files:
  -source: /
   destination: /home/ec2-user/app/step2/zip/
   overwrite: yes
```

- version: 0.0
1. CodeDeploy 버전을 이야기함
2. 프로젝트 버전이 아니므로 0.0 외에 다른 버전을 사용하면 오류 발생

- source
1. CodeDeploy에서 전달해 준 파일 중 destination으로 이동시킬 대상을 지정
2. 루트 경로(/)를 지정하면 전체 파일을 이야기함

- destination
1. source에서 지정된 파일을 받을 위치
2. 이후 Jar를 실행하는 등은 destination에서 옮긴 파일들로 진행

- overwrite
1. 기존에 파일들이 있으면 덮어쓸지를 결정
2. 현재 yes라고 했으니 파일을 덮어쓰게 됨

### 9.5 배포 자동화 구성

```bash
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl springboo2-webservice | grep jar | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRNET_PID"

if[ -z "$CURRNET_PID" ]; then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다"
else
  echo "> kill -15 $CURRNET_PID"
  kill -15 $CURRENT_PID
  sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME에 실행권한 추가"

chomod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
  -Dspring.config.location=classpath:/application.properties,classpath:/application-real.proerties,
/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
  -Dspring.profiles.active=real \
  $JAR_NAME > $REPOSITORY/nohup.out 2>&1
```

- CURRNET_PID
1. 현재 수행 중인 스프링 부트 애플리케이션의 프로세스 ID를 찾음
2. 실행 중이면 종료하기 위해서임

- chmod +x $JAR_NAME
1. Jar 파일은 실행 권한이 없는 상태
2. nohup으로 실행할 수 있게 실행 권한을 부여

- $JAR_NAME >$REPOSITORY/nohup.out 2>&1 &
1. nohup 실행 시 CodeDeploy는 무한 대기
2. 이 이슈를 해결하기 위해 nohup.out 파일을 표준 입출력용으로 별도로 사용
3. 이렇게 하지 않으면 nohup.out 파일이 생기지 않고, CodeDeploy 로그에 표준 입출력이 출력됨
4. nohup이 끝나기 전까지 CodeDeploy도 끝나지 않으니 꼭 이렇게 해야 함

**.travis.yml 파일 수정**

```bash
before_deploy:
  -mkdir -p before-deploy #zip에 포함시킬 파일들을 담을 디렉토리 생성
  -cp scripts/*.sh before-deploy/
  -cp appspec.yml before-deploy/
  -cp build/libs/*.jar before-deploy/
  -cd before-deploy && zip -r before-deploy * #before-deploy로 이동 후 전체 압축
  -cd ../ && mkdir -p deploy #상위 디렉토리로 이동 후 deploy 디렉토리 생성
  -mv before-deploy/before-deploy.zip deploy/springboo2-webservice.zip #deploy로 zip 파일이동
```

1. Travis CI는 S3로 특정 파일로만 업로드가 안됨
    - 디렉토리 단위로만 업로드 할 수 있기 때문에 before-deploy 디렉토리는 항상 생성함
2. before-deploy에는 zip 파일에 포함시킬 파일들을 저장
3. zip -r 명령어를 통해 before-deploy 디렉토리 전체 팡리을 압축

**appspec.yml 파일 수정**

```bash
permissions:
  -object: /
   pattern: "**"
   owner: ec2-user
   group: ec2-iser

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
```

- permissions
1. CodeDeploy에서 EC2 서버로 넘겨준 파일들을 모두 ec2-user 권한을 갖도록 함

- hooks
1. CodeDeploy 배포 단계에서 실행할 명령어를 지정
2. ApplicationStart라는 단계에서 deploy.sh를 ec2-user 권한으로 실행하게 함
3. timeout: 60으로 스크립트 실행 60초 이상 수행되면 실패가 됨
