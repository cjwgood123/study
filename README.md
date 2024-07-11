# study
study


SpringBoot Gradle Thymeleaf 로 구성된 마이페이지 프로젝트를 신규로 생성하기로 했다.

이전 진행하던 프로젝트를 기반으로 신규로 프로젝트를 생성하기로 했다.

원래 , 회사에 구성되어있던 

CentOs 서버 내 
Docker 가 있었고 , 해당 도커내 Jenkins 가 Deploy 되어있었다.
나는 신규로 프로젝트를 생성해서 JenKins PipeLine을 작성하여 Docker에 

이건 해당 jenkins 파일인데 
pipeline {
    agent any
     tools {
         jdk("jdk1.8")
     }
    environment {
        BUILD_TARGET_HOME = '${BUILD_HOME}'
        PROJECT = 'denb_boot'
        APP_WAR_FILE = 'denb_boot.war'
    }

    stages {
        stage('빌드 테스트') {
             steps {
                  echo 'Build Start "${PROJECT}"'
                  sh 'chmod +x gradlew'
                  sh './gradlew bootWar'
                  echo 'Build End "${PROJECT}"'
              }
        }
        stage('원격 서버로 복사') {
          steps {
              echo 'Copy Start "war파일 이동"'
              sh'''
              scp -i /var/jenkins_home/key/denb-v1.pem -P 38371 -r ./docker dena@58.229.180.228:/home/dena/denb_boot
              cd ./${APP_API}/build/libs
              scp -i /var/jenkins_home/key/denb-v1.pem -P 38371 ${APP_WAR_FILE} dena@58.229.180.228:/home/dena/denb_boot/docker/denb_boot
              '''
              echo 'Copy End "war파일 이동"'
          }
        }
        stage('실행') {
          steps {
              echo 'Execute Start "${PROJECT}"'

              sh'''
              ssh -i /var/jenkins_home/key/denb-v1.pem -p 38371 -t dena@58.229.180.228 -T "cd /home/dena/denb_boot/docker/denb_boot ; bash;" "chmod +x docker-web-deploy.sh"
              ssh -i /var/jenkins_home/key/denb-v1.pem -p 38371 -t dena@58.229.180.228 -T "cd /home/dena/denb_boot/docker/denb_boot ; bash;" "./docker-web-deploy.sh  denb_boot"
              '''
              echo 'Execute End "${PROJECT}"'
          }
        }
    }
    post {
        success {
            echo "SUCCESS"
        }
    }
}
실제 오류는 war 파일을 실행시키는 sh 파일을 배포하지 않아서 발생했지만 , 오류는 권한 문제라고 나와서 애를 먹었었다........................

java -Ds pring.profiles.active=local -Djava.security.egd=file:/dev/./urandom -jar /$COPY_WAR_FILE  (start.sh파일)
젠장....

pipeLine 과 jenkins , gitWebhook 을 설정해서 하는 작업들은 재미있었다. 

그치만...........사내 회의 후에 war 배포 혹은 jar 배포를 진행하는것으로 바뀌었다........!!!!


대충의 구조는 
![image](https://github.com/cjwgood123/study/assets/47588727/7648ba94-7f71-4042-8318-9b56ba5eb024)

이러한 구조를 지니고 있었다.


나중에 컴포즈 파일 찾기 귀찮을때 참고해야겠다.

# redis/conf/redis.conf 
파일 규격 버전
version: "3.1"

실행하려는 컨테이너들 정의
services:
  web: 
    container_name: denb_boot
    hostname: denb_boot
    ports:
      - "8080:8080"
    env_file:
      - ./denb_boot/config/.env.dev
    build:
      context: ./denb_boot/
    deploy:
      mode: replicated
      replicas: 1
    networks:
      - network

networks: # 가장 기본적인 bridge 네트워크
  network:
    driver: bridge

엔진엑스 , 레디스를 도커 이미지로 개발서버에 띄워져 있었는데 , 도커로 img를 띄워놓고 포트만 설정해놓은 가장 기본세팅인것 같았다. 추후에 Spring boot 에도 redis를 적용시킬거니까 나중에 참고해야겠음!



 어떤 네트위크 인터페이스로부터 연결할 수 있도록 할 것인지 관리 (여기에서는 Anywhere)
bind 0.0.0.0

사용 포트 관리
port 6379

Master 노드의 기본 사용자(default user)의 비밀번호 설정
requirepass seed2804

Redis 에서 사용할 수 있는 최대 메모리 용량. 지정하지 않으면 시스템 전체 용량
maxmemory 2gb

 maxmemory 에 설정된 용량을 초과했을때 삭제할 데이터 선정 방식
 - noeviction : 쓰기 동작에 대해 error 반환 (Default)
 - volatile-lru : expire 가 설정된 key 들중에서 LRU algorithm 에 의해서 선택된 key 제거
 - allkeys-lru : 모든 key 들 중 LRU algorithm에 의해서 선택된 key 제거
 - volatile-random : expire 가 설정된 key 들 중 임의의 key 제거
 - allkeys-random : 모든 key 들 중 임의의 key 제거
 - volatile-ttl : expire time(TTL)이 가장 적게 남은 key 제거 (minor TTL)
maxmemory-policy volatile-ttl

  # DB 데이터를 주기적으로 파일로 백업하기 위한 설정입니다. Redis 가 재시작되면 이 백업을 통해 DB 를 복구합니다.

 15분 안에 최소 1개 이상의 key 가 변경 되었을 때 save 900 1
 5분 안에 최소 10개 이상의 key 가 변경 되었을 때 save 300 10
 60초 안에 최소 10000 개 이상의 key 가 변경 되었을 때 save 60 10000

이건 엔진엑스  컨피그 파일인데 실제로 사용해보지 못한게 아쉽다.

user  nginx;
worker_processes  auto;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        # nginx를 통해 외부로 노출되는 port.
        listen 80;

        location / {

            # arbitrary한 upstream명
            proxy_pass         http://web/;
        }
    }

    upstream web{
        server patient-care-web01:8080 weight=100 max_fails=100 fail_timeout=3s;
        server patient-care-web02:8080 weight=100 max_fails=100 fail_timeout=3s;
    }

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    keepalive_timeout  65;
    include /etc/nginx/conf.d/default.conf;
}


