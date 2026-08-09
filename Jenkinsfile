pipeline {
    agent any // 어떤 에이전트 실행서버든 실행가능

    tools { // tools : jenkins tools에 등록된 도구 사용
        maven 'maven 3.9.12'  // 젠킨스 툴에 등록한 이름과 일치해야함
    }

    environment { // environment : 환경변수 정의
        // JAVA_HOME = tool name: 'jdk 11.0.20', type: 'jdk' // jenkins tools에 등록된 jdk 사용
        // PATH = "${JAVA_HOME}/bin:${PATH}" // 환경변수 PATH에 JAVA_HOME/bin 추가
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
        stage('Git Checkout') { // 수행 단계 구분
            steps { // steps : 각 단계에서 수행할 작업 정의
                // git branch: 'main', url: 
                // sh 'mvn clean package' // 쉘 명령어 실행
                // jenkins가 연결된 git 저장소에서 최신코드 checkout
                checkout scm
                
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests' // 쉘 명령어 실행
            }

        stage('Prepare Jar') {
            steps {
                // 빌드 결과물을 app.jar로 복사
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}' // 빌드된 jar 파일을 app.jar로 복사
                // sh 'cp target/*.jar ${JAR_FILE_NAME}' // 빌드된 jar 파일을 app.jar로 복사
            }
        }

        stage('Copy Jar to Remote Server') {
            steps {
                // 원격 명령 실행
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
                    // 배포 디렉토리 생성 (없으면 생성)
                    sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                    // JAR 파일과 Dockerfile을 원격 서버로 전송
                    sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"

                    // 원격 서버로 jar 파일 전송
                    // sh "scp -i /var/lib/jenkins/.ssh/id_rsa ${JAR_FILE_NAME} ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/${JAR_FILE_NAME}"
                }
            }
        }
    }
}

}