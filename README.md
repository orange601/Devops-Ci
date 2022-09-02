# devops-ci
빌드/테스트 자동화

## 목표 ##
1. github webhook 설정
2. jenkins를 통한 build
3. Was가 인식 할 수 있는 volume에 build 된 jar 배포

## Jenkins ##
- Docker안에 Jenkins를 설치한다.

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
      - //var/run/docker.sock:/var/run/docker.sock
    user: root
    privileged: true

volumes:
    볼륨명:

networks:
  default:
    external:
      name: was-network
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

### 2. ngrok 설정 ###
- ngrok.exe http 8080
- **방화벽 뒤에있는 사내 로컬 서버를 안전한 터널을 통해 공개 인터넷에 노출할 수 있도록 지원해주는 플랫폼**
- [윈도우환경에서 설치 및 사용방법 출처:프뚜](https://ssjeong.tistory.com/entry/ngrok-%EB%A1%9C%EC%BB%AC-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EC%9D%98-%ED%84%B0%EB%84%90-%EC%97%B4%EA%B8%B0%EB%A1%9C%EC%BB%AC-PC-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95)
- 서버 재부팅시 URL 변경된다.
- connection이 20으로 한정된다.


### 3. jenkins Plugin ###
- gradle로 빌드시 Invoke Gradle script 기능을 사용해야하므로 plugin에서 gradle을 설치해야만 사용할 수 있다.

### 4. docker-compose 설치 ###
- sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- chmod +x /usr/local/bin/docker-compose
- ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
## 서버 재부팅 ##
- ngrok 실행 여부
- rgrok이 재 실행되면 URL이 변경된다. webhook URL 확인

### 주의사항 ###
1. 웹훅
    - http://locahost:8080를 입력하시면 정상적으로 동작하지 않습니다.
    - http://public-ip:8080 같이 공개 IP를 사용하는 경우에도 정상적으로 동작하지 않습니다.
    - ngrok 어플리케이션을 통해 외부에서 접근할 수 있는 도메인을 사용합니다.
    - Content type - application/json 타입을 사용합니다.
