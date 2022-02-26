### 10.1 무중단 배포 소개

무중단 배포

- 서비스를 정지하지 않고 배포할 수 있는 방법
1. AWS에서 블루 그린 무중단 배포
2. 도커를 이용한 웹서비스 무중단 배포

엔진엑스

- 웹 서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍 등을 위한 오픈소스 소프트웨어
- 리버스 프록시: 엔진엑스가 외부의 요청을 받아 백엔드 서버로 요청을 전달하는 행위
- 저렴함

**엔진엑스 무중단 배포1**

![11](https://user-images.githubusercontent.com/57864944/155552873-86768f8c-cf26-4392-9ea3-391f86c79696.png)

1. 사용자는 서비스 주소로 접속(80 혹은 443 포트)
2. 엔진엑스는 사용자의 요청을 받아 현재 연결된 스프링 부트로 요청을 전달
3. 스프링 부트2는 엔진엑스와 연결된 상태가 아니니 요청받지 못함

⇒ 1.1 버전으로 신규 배포가 필요하면, 엔진엑스와 연결되지 않은 스프링 부트2로 배포

**엔진엑스 무중단 배포2**

![22](https://user-images.githubusercontent.com/57864944/155552941-855351a3-772e-4507-8c59-a55b593a6e73.png)
1. 배포하는 동안에도 서비스는 중단 X
2. 배포가 끝나고 정상적으로 스프링부트2가 구동중인지를 확인
3. 스프링부트2가 정상적으로 구동 중이면 nginx reload 명령어를 통해 8081 대신 8082를 바라보도록 함
4. nginx reload는 0.1초 이내에 완료됨

**엔진엑스 무중단 배포3**

![33](https://user-images.githubusercontent.com/57864944/155552948-a7eac859-aeb3-405f-9c6a-6083b51618e9.png)
1. 현재는 엔진엑스와 연결된 것이 스프링부트2
2. 스프링 부트1의 배포가 끝났다면 엔진엑스가 스프링 부트1을 바라보게 변경하고 nginx reload를 실행함
3. 이후 요청부터는 엔진엑스가 스프링 부트1로 요청을 전달함

![44](https://user-images.githubusercontent.com/57864944/155552954-84b3680f-60a4-441a-bc57-10194ecceeda.png)

### 10.3 무중단 배포 스크립트 만들기

```java
@RequiredArsgConstructor
@RestController
public class ProfileController{
    private final Environment env;
    @GetMapping("/profile")
    public String profile(){
        List<String> profiles=Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles=Arrays.asList("real","real1","real2");
        String defaultProfile=profiles.isEmpty()! "default":profiles.get(0);
        return profiles.stream()
                       .filter(realProfiles::contains)
                       .findAny()
                       .orElse(defaultProfile);
    }
}
```

- env.getActiveProfiles()
1. 현재 실행 중인 ActiveProfile을 모두 가져옴
2. 즉, real, oauth, real-db 등이 활성화되어 있다면 3개 모두 담겨 있음
3. 여기서 real, real1, real2는 모두 배포에 사용될 profile이라 이 중 하나라도 있으면 그 값을 반환하도록 함

**배포 스크립트들 작성**

- stop.sh: 기존 엔진엑스에 연결되어 있진 않지만, 실행 중이던 스프링 부트 종료
- start.sh: 배포할 신규 버전 스프링 부트 프로젝트를 stop.sh로 종료한 profile로 실행
- health.sh: start.sh로 실행시킨 프로젝트가 정상적으로 실행됐는지 체크
- switch.sh: 엔진엑스가 바라보는 스프링 부트를 최신 버전으로 변경
- profile.sh: 앞선 4개 스크립트 파일에서 공용으로 사용할 profile과 포트 체크 로직

profile.sh

```bash
#!/user/bin/env bash

#쉬고 있는 profile 찾기: real1이 사용 중이면 real2가 쉬고 있고, 반대편 real1이 쉬고있음
function find_idle_profile(){
  RESPONSE_CODE=$(curl -s -o /dev/null -w "%(http_code)" http://localhost/profile)
  if [${RESPONCE_CODE} -ge 400] #400 보다 크면
  then
    CURRNET_PROFILE=real2
  else
    CURRENT_PROFILE=$(curl -s http://localhost/profile)
  fi
  
  if [${CURRENT_PROFILE}==real1]
  then
    IDLE_PROFILE=real2
  else
    IDLE_PROFILE=real1
  fi

  echo "${IDLE_PROFILE}"
}

#쉬고 있는 profile의 port 찾기
function find_idle_port(){
  IDLE_PROFILE=$(find_idle_profile)
  if[${IDLE_PROFILE}==real1]
  then
    echo "8081"
  else
    echo "8082"
  fi
}
```

- $(curl -s -o /dev/null -w “%{http_code}” http://localhost/profile)
1. 현재 엔진엑스가 바라보고 있는 스프링 부트가 정상적으로 수행 중인지 확인
2. 응답값을 HttpStatus로 받음
3. 정상이면 200, 오류가 발생한다면 400~503 사이로 발생하니 400이상은 모두 예외로 함
4. real2를 현재 profile로 사용

- IDLE_PROFILE
1. 엔진엑스와 연결되지 않은 profile
2. 스프링 부트 프로젝트를 이 profile로 연결하기 위해 반환

- echo “${IDLE_PROFILE}”
1. bash라는 스크립트는 값을 반환하는 기능이 없음
2. 그래서 제일 마지막 줄에 echo로 결과를 출력 후, 클라이언트에서 그 값을 잡아서 사용
3. 중간에 echo를 사용해서는 안됨

stop.sh

```bash
#!/user/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동 중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [-z ${IDLE_PID}]
then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다"
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```

- ABSDIR=$(dirname $ABSPATH)
1. 현재 stop.sh가 속해 있는 경로를 찾음
2. 하단의 코드와 같이 profile.sh의 경로를 찾기 위해 사용

- source ${ABSDIR}/profiles.sh
1. 자바로 보면 일종의 import 구문
2. 해당 코드로 인해 stop.sh에서도 profile.sh의 여러 function을 사용할 수 있게 됨

start.sh

```bash
#!/usr/bin/evn bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=springboot2-webservice

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 애플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME에 실행권한 추가"

chomod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config/location=classpath:/application.properties,classpath:/appilcation-$IDLE_PROFILE.properties,/
home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real.db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

1. 기본적인 스크립트는 step2의 deploy.sh와 유사
2. 다른점은 IDLE_PROFILE을 통해 properties 파일을 가져오고, active profile을 지정하는 것
3. 여기서도 IDLE_PROFILE을 사용하니 profile.sh를 가져와야함

health.sh

```bash
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR-$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Checek Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile"
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDEL_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -1)
 
  if [ ${UP_COUNT} -ge 1]
  then # $up_count >=1 ("real" 문자열이 있는지 검증)
    echo "> Health check 성공"
    switch_proxy
    break
  else
    echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
    echo "> Health check: ${RESPONSE}"
  fi

  if [${RETRY_COUNT} -eq 10]
  then
    echo "> Health check 실패."
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi
  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

1. 엔진엑스와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지 체크
2. 잘 떴는지 확인 되어야 엔진엑스 프록세 설정을 변경함
3. 엔진엑스 프록시 설정 변경은 switch.sh에서 수행

switch.sh

```bash
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy(){
  IDLE_PORT=$(find_idle_port)
 
  echo "> 전환할 Port: $IDLE_PORT"
  echo "> Port 전환"  
  echo "set \$service_url http://127.0.0.1:${IDLE_PORT}; | sudo tee /ect/nginx/conf.d/service-url.inc
  echo "> 엔진엑스 Reload"
  sudo service nginx reload
}
```

- echo “set \$service_url http://127.0.0.1:${IDLE_PORT};
1. 하나의 문장을 만들어 파이프라인으로 넘겨주기 위해 echo 사용
2. 엔진엑스가 변경할 프록시 주소를 생성
3. 쌍따옴표를 사용
4. 사용하지 않으면 $service_url을 그대로 인식하지 못하고 변수를 찾게됨

> sudo tee /etc/nginx/conf.d/service-url.inc
> 
1. 엔진엑스 설정을 다시 불러옴
2. restart와는 다름
3. restart는 잠시 끊기는 현상이 있지만 reload는 끊김 없이 다시 불러옴
4. 다만, 중요한 설정들은 반영되지 않으므로 restart를 사용
5. 여기선 외부의 설정 파일인 service-url을 다시 불러오는거라 reload로 가능함
