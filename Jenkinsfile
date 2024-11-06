pipeline {
  agent any
  environment {
    DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'  // 추가한 Credentials ID
  }
  stages {
    stage('Git SCM Update') {
      steps {
        git url: 'https://github.com/paperdoll96/jenkinsTest.git', branch: 'master'
      }
    }
    stage('Docker Build and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
          echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
          docker build -t paperdoll96/keduitlab:white .
          docker push paperdoll96/keduitlab:white
          '''
        }
      }
    }
    stage('Ansible: Pull Image on Nodes') {
      steps {
        sh '''
        ansible node -b -m shell -a "docker pull paperdoll96/keduitlab:white"
        '''
      }
    }
    stage('Ansible: Kubernetes Deploy and Service on Master') {
      steps {
        sh '''
        ansible master -b -m shell -a "
          sudo kubectl create deployment jenkinstest --replicas 3 --port=80 --image=paperdoll96/keduitlab:white
          sudo kubectl expose deployment jenkinstest --type=LoadBalancer --name=jenkinstest-service --port=80 --target-port=80
        "
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
