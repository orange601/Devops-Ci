# devops-ci
빌드/테스트 자동화

## CI CD 도구 ##
1. Github actions: Github에서 제공하는 워크플로우(workflow)를 자동화하도록 도와주는 도구
- github actions에서 파일전달시 scp를 이용해서 파일전달을 해야 하는데 scp는 리눅스만 지원된다고 한다.

2. Travis CI: 유료 ( 무료버전도 천원정도 결제 된다고 한다 )

3. CodeDeploy: 자동 배포하는 역할을 수행하는 AWS Service

## 목표 ##
1. github webhook 설정
2. jenkins를 통한 build
3. Was가 인식 할 수 있는 volume에 build 된 jar 배포

## Jenkins ##

### 1. 설치 ###
- **jenkins 설치시 jdk설치가 되어있어야 하므로 jenkins:jdk11 설치 한다.**
**docker-compose.yml**
````yml
version: '3.1'

services:
  jenkins:
    build: .
    #image: jenkins/jenkins:jdk11
    container_name: v2_jenkins
    restart: on-failure
    ports:
      - "20000:8080"
    volumes:
      - 볼륨명:/sharing/
      - /var/lib/docker/volumes/볼륨명/_data/jenkins/home/:/var/jenkins_home/
      - //var/run/docker.sock:/var/run/docker.sock # container 안에서 host의 docker 명령어를 사용 가능
    user: root
    privileged: true
    extra_hosts: # jenkins 안에서 localhost 와 host의 localhost는 다르다 host의 localhost를 연결해주기 위한 설정이다.
      - host.docker.internal:host-gateway

volumes:
    볼륨명:

networks:
  default:
    name: was-network
    external: true
      
````
**Dockerfile**
````
FROM jenkins/jenkins:jdk11
USER root
RUN curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose
RUN apt-get update && \
    apt-get -y install apt-transport-https \
      ca-certificates \
      curl \
      gnupg2 \
      software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
    add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
      $(lsb_release -cs) \
      stable" && \
   apt-get update && \
   apt-get -y install docker-ce
````

### 2. 젠킨스 이동/복사 ###
- 서버 이전을 한다거나 새로 세팅하는 등 젠킨스를 옮겨야 하는 경우가 있다. 
- 운영 중인 job이 있기 때문에 Credential 이나 Fingerprint, Plugin 등 기존 제킨스 설정을 그대로 함께 옮겨야 한다.
- 젠킨스는 DB 같은 별도의 스토리지를 사용하지 않고, 폴더와 파일을 사용한다. 
- JOB 이름으로 폴더를 생성하고, 그 안에 JOB 설정 파일과 실행 히스토리 등을 별도로 저장한다. 이 파일들만 잘 복사하면 젠킨스는 그대로 복사가 된다.

1. 새 젠킨스를 설치한다. (버전업을 해도 좋다.)
2. JENKINS_HOME 하위에 있는 파일들을 새 젠킨스로 복사한다.
3. 새 젠킨스를 실행한다.
[출처](https://blog.leocat.kr/notes/2017/11/02/jenkins-backup-restore)

** docker jenkins 설치시 host의 jenkins 경로와 local의 /var/jenkins_home 의 Volume을 맞춰두면 된다. **
** 예) **
````yml
    volumes:
      - orange/jenkins/:/var/jenkins_home/
````

### 3. ngrok 설정 ###
- **방화벽 뒤에있는 사내 로컬 서버를 안전한 터널을 통해 공개 인터넷에 노출할 수 있도록 지원해주는 플랫폼**
- 프로그램 재 시작 시 (서버 재부팅 시) URL 변경된다.
- CI툴(JENKINS)에서 사용할 포트 방화벽을 열어야하므로 CI툴의 PORT를 열어준다,
- 비회원은 세션이 8시간으로 한정되어있기때문에 회원가입 후 토큰을 등록한다.

### 4. Webhook 설정 ###
- JENKINS- Gradle 설정
  1. Jenkins 관리에서 Global Tool Configuration 클릭
  2. 스크롤을 내려 Gradle 설정, 이름과 프로젝트의 Gradle 버전 선택 후 Save
  3. 프로젝트 설정에서 Build Steps의 Invoke Gradle script 설정시 위에서 추가한 Gradle을 선택
  4. Tasks 입력

- Build Steps
````
clean
build --stacktrace
````

- Execute shell
````
bash /sharing/bash/deploy.sh
````

### 주의사항 ###
1. 웹훅
    - http://locahost:8080를 입력하시면 정상적으로 동작하지 않습니다.
    - http://public-ip:8080 같이 공개 IP를 사용하는 경우에도 정상적으로 동작하지 않습니다.
    - ngrok 어플리케이션을 통해 외부에서 접근할 수 있는 도메인을 사용합니다.
    - Content type - application/json 타입을 사용합니다.

## Trouble Shooting ##
### 1. 웹훅  ###
- ngrok에 url과 girhub에서 설정한 webhook URL이 같은지 확인

