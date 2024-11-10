pipeline {
  agent any

  environment {
    HARBOR_CREDENTIALS_ID = 'harbor-credentials'
    HARBOR_URL = '211.183.3.198'
    HARBOR_REPO = 'myproject/keduitlab'
    SPRING_BOOT_REPO = 'https://github.com/paperdoll96/rrs.git'
  }

  stages {
    stage('Clone Spring Boot Repository') {
      steps {
        git url: "${SPRING_BOOT_REPO}", branch: 'main'
        // Spring Boot 애플리케이션의 최신 소스 코드 가져오기
      }
    }

    stage('Build Spring Boot Application') {
      steps {
        sh './gradlew bootJar'
        // Spring Boot 애플리케이션 빌드
      }
    }

    stage('Docker Build and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${HARBOR_CREDENTIALS_ID}", usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
          sh '''
          echo $HARBOR_PASS | docker login $HARBOR_URL -u $HARBOR_USER --password-stdin
          docker build -t $HARBOR_URL/$HARBOR_REPO:springboot .
          docker push $HARBOR_URL/$HARBOR_REPO:springboot
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
        ansible-playbook playbook.yml
        '''
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
