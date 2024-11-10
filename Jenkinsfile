pipeline {
    agent any

    environment {
        HARBOR_CREDENTIALS_ID = 'harbor-credentials'  // Jenkins에 설정된 Harbor 자격증명 ID
        HARBOR_URL = '211.183.3.198'                  // Harbor 서버 URL
        HARBOR_REPO = 'myproject/keduitlab'           // Harbor 프로젝트와 이미지 경로
    }

    stages {
        stage('Clone Repositories') {
            steps {
                script {
                    // Spring Boot 애플리케이션 클론
                    dir('rrs') {
                        git url: 'https://github.com/paperdoll96/rrs.git', branch: 'master'
                    }
                    // CI/CD 및 인프라 설정 클론
                    dir('jenkinsTest') {
                        git url: 'https://github.com/paperdoll96/jenkinsTest.git', branch: 'master'
                    }
                }
            }
        }

        stage('Build Spring Boot Application') {
            steps {
                script {
                    dir('rrs') { // Spring Boot 디렉토리에서 작업
                        if (isUnix()) {
                            sh 'chmod +x ./gradlew' // Unix/Linux 환경에서 실행 권한 추가
                        }
                        sh './gradlew bootJar' // 빌드 실행
                    }
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${HARBOR_CREDENTIALS_ID}", usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                    script {
                        sh '''
                        echo $HARBOR_PASS | docker login $HARBOR_URL -u $HARBOR_USER --password-stdin
                        docker build -t $HARBOR_URL/$HARBOR_REPO:springboot ./rrs
                        docker push $HARBOR_URL/$HARBOR_REPO:springboot
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('jenkinsTest') { // 인프라 설정 디렉토리에서 Playbook 실행
                        sh 'ansible-playbook playbook.yml'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment and service creation successful on master with 3 replicas!'
        }
        failure {
            echo 'Deployment failed. Check the logs for details.'
        }
    }
}
