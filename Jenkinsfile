pipeline {
    agent any // 어떤 에이전트 실행서버든 실행가능

    tools { // tools : jenkins tools에 등록된 도구 사용
        maven 'maven 3.9.12'  // 젠킨스 툴에 등록한 이름과 일치해야함
    }

    environment { // environment : 환경변수 정의
        DOCKER_IMAGE = "demo-app" // 빌드번호를 태그로 사용
        CONTAINER_NAME = "springboot-container" // 컨테이너 이름
        JAR_FILE_NAME = "app.jar" // 빌드된 jar 파일 경로
        PORT = "8081" // 컨테이너에서 노출할 포트

        REMOTE_USER = "ec2-user"
        REMOTE_HOST = "3.36.217.169"

        REMOTE_DIR = "/home/ec2-user/deploy" // 원격 서버에서의 디렉토리 경로
        SSH_CREDENTIALS_ID = "fbf91916-ded8-41ea-897f-df77393be06f" // Jenkins
    }

    stages { // stages : 빌드, 테스트, 배포 등 단계별로 정의
        stage('Git Checkout') { 
            steps { 
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests' 
            }
        } // <-- 누락되었던 stage 중괄호 추가!

        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}' 
            }
        }

        stage('Copy Jar to Remote Server') {
            steps {
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                    sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
                }
            }
        }

        stage('Remote Docker Build & Deploy') {
            steps {
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
                    // sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"cd ${REMOTE_DIR} && docker build -t ${DOCKER_IMAGE} . && docker stop ${CONTAINER_NAME} || true && docker rm ${CONTAINER_NAME} || true && docker run -d --name ${CONTAINER_NAME} -p ${PORT}:8080 ${DOCKER_IMAGE}\""
                    ss """
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << ENDSSH
    cd ${REMOTE_DIR} || exit 1
    docker rm -f ${CONTAINER_NAME} || true
    docker build -t ${DOCKER_IMAGE} .
    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}
ENDSSH
                    """
                
                }
            }
        }
    }
}