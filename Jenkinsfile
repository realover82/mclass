pipeline {
    agent any // 어떤 에이전트 실행서버든 실행가능

    tools { // tools : jenkins tools에 등록된 도구 사용
        maven 'maven 3.9.12'  // 젠킨스 툴에 등록한 이름과 일치해야함
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
    }
}

