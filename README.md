# devops-ci
빌드/테스트 자동화

## 목표 ##
1. jenkins를 통한 webhook 빌드
2. sonarqube나 lint를 통한 테스트
3. slack 알림

## jenkins 설정 ##
- Webhook 설정
- slack 설정

### 1. 설치 ###
- **jenkins 설치시 jdk설치가 되어있어야 하므로 jenkins:jdk11 설치 한다.**
````yml
version: '3.1'

services:
  jenkins:
    image: jenkins/jenkins:jdk11
    container_name: jenkins
    ports:
      - "20000:8080"
    volumes:
      - ../volume:/var/jenkins_home
    user: root
    privileged: true

networks:
  default:
    external:
      name: was-network
````

### 2. ngrok 설정 ###
- ngrok.exe http 8080
- **방화벽 뒤에있는 사내 로컬 서버를 안전한 터널을 통해 공개 인터넷에 노출할 수 있도록 지원해주는 플랫폼**
- [윈도우환경에서 설치 및 사용방법 출처:프뚜](https://ssjeong.tistory.com/entry/ngrok-%EB%A1%9C%EC%BB%AC-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EC%9D%98-%ED%84%B0%EB%84%90-%EC%97%B4%EA%B8%B0%EB%A1%9C%EC%BB%AC-PC-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95)
- 서버 재부팅시 URL 변경된다.
- connection이 20으로 한정된다.


### 3. jenkins Plugin ###
- gradle로 빌드시 Invoke Gradle script 기능을 사용해야하므로 plugin에서 gradle을 설치해야만 사용할 수 있다.

## 서버 재부팅 ##
- ngrok 실행 여부
- rgrok이 재 실행되면 URL이 변경된다. webhook URL 확인
