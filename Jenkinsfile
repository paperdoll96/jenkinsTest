pipeline {
  agent any
  environment {
    HARBOR_CREDENTIALS_ID = 'harbor-credentials'  // Jenkins에 추가한 Harbor 자격증명 ID
    HARBOR_URL = '211.183.3.198'          // Harbor HTTPS URL
    HARBOR_REPO = 'myproject/keduitlab'           // Harbor에서 생성한 프로젝트 이름 + 레포 경로
  }
  stages {
    stage('Git SCM Update') {
      steps {
        git url: 'https://github.com/paperdoll96/jenkinsTest.git', branch: 'master'
      }
    }
    stage('Docker Build and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${HARBOR_CREDENTIALS_ID}", usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
          sh '''
          echo $HARBOR_PASS | docker login $HARBOR_URL -u $HARBOR_USER --password-stdin
          docker build -t $HARBOR_URL/$HARBOR_REPO:white .
          docker push $HARBOR_URL/$HARBOR_REPO:white
          '''
        }
      }
    }
    stage('Ansible: Pull Image on Nodes') {
      steps {
        sh '''
        ansible node -b -m shell -a "docker pull $HARBOR_URL/$HARBOR_REPO:white"
        '''
      }
    }
    stage('Ansible: Kubernetes Deploy and Service on Master') {
      steps {
        sh '''
        ansible master -b -m shell -a "kubectl create deployment jenkinstest --replicas=3 --port=80 --image=$HARBOR_URL/$HARBOR_REPO:white && \
                                       kubectl expose deployment jenkinstest --type=LoadBalancer --name=jenkinstest-service --port=80 --target-port=80"
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
