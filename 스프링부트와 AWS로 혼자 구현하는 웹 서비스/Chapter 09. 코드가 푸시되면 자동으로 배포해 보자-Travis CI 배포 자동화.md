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
