pipeline {
  agent any
  stages {
    stage('Git SCM Update') {
      steps {
        git url: 'https://github.com/paperdoll96/jenkinsTest.git', branch: 'master'
      }
    }
    stage('Docker Build and Push') {
      steps {
        sh '''
        sudo docker build -t paperdoll96/keduitlab:purple .
        sudo docker push paperdoll96/keduitlab:purple
        '''
      }
    }
    stage('Kubernetes Deploy and Service') {
      steps {
        sh '''
        # 기존 Deployments 및 Services 삭제 (필요 시)
        sudo kubectl delete deployment keduitlab-deployment || true
        sudo kubectl delete svc keduitlab-service || true

        # 새로운 Deployment 생성
        sudo kubectl create deployment keduitlab-deployment --image=paperdoll96/keduitlab:purple

        # Service 생성 (LoadBalancer)
        sudo kubectl expose deployment keduitlab-deployment --type=LoadBalancer --name=keduitlab-service --port=80 --target-port=80
        '''
      }
    }
  }
  post {
    success {
      echo 'Deployment and service creation successful!'
    }
    failure {
      echo 'Deployment failed. Check the logs for details.'
    }
  }
}
