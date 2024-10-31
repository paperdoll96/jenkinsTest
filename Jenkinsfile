pipeline {
  agent any
  stages {
    stage('git scm update') {
      steps {
        git url: 'https://github.com/paperdoll96/jenkinsTest.git', branch: 'master'
      }
    }
    stage('docker build and push') {
      steps {
        sh '''
        sudo docker build -t paperdoll96/keduitlab:pupple .
        sudo docker push paperdoll96/keduitlab:pupple
        '''
      }
    }
    stage('deploy and service') {
      steps {
        sh '''
        sudo kubectl apply -f test.yml
        '''
      }
    }
  }
}
