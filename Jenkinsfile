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
        sudo docker build -t paperdoll96/keduitlab:white .
        sudo docker push paperdoll96/keduitlab:white
        '''
      }
    }
    stage('Ansible: Pull Image on Nodes') {
      steps {
        sh '''
        # Ansible을 사용해 node1, node2, node3에서 Docker 이미지를 풀링
        ansible node -m shell -a "docker pull paperdoll96/keduitlab:white"
        '''
      }
    }
    stage('Ansible: Kubernetes Deploy and Service on Master') {
      steps {
        sh '''
        # Ansible을 사용해 master 노드에서 Kubernetes 배포 및 서비스 생성
        ansible master -m shell -a "
          kubectl create deployment jenkinstest --replicas 3 --port=80 --image=paperdoll96/keduitlab:white;
          kubectl expose deployment jenkinstest --type=LoadBalancer --name=jenkinstest-service --port=80 --target-port=80;
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
