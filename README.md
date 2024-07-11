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
