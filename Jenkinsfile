pipeline {
  agent any  // Jenkins 파이프라인을 어떤 에이전트에서나 실행 가능

  environment {
    HARBOR_CREDENTIALS_ID = 'harbor-credentials'  // Jenkins에 설정된 Harbor 자격증명 ID
    HARBOR_URL = '211.183.3.198'                  // Harbor 서버 URL
    HARBOR_REPO = 'myproject/keduitlab'           // Harbor 프로젝트와 이미지 경로
  }

  stages {
    stage('Git SCM Update') {  // 최신 소스 코드를 GitHub에서 가져오는 단계
      steps {
        git url: 'https://github.com/paperdoll96/jenkinsTest.git', branch: 'master'
        # GitHub의 master 브랜치에서 Jenkinsfile 및 관련 소스 코드 다운로드
      }
    }

    stage('Docker Build and Push') {  // Docker 이미지 빌드 및 Harbor에 Push
      steps {
        withCredentials([usernamePassword(credentialsId: "${HARBOR_CREDENTIALS_ID}", usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
          # Harbor 자격증명을 사용하여 로그인
          sh '''
          echo $HARBOR_PASS | docker login $HARBOR_URL -u $HARBOR_USER --password-stdin
          # Harbor에 로그인

          docker build -t $HARBOR_URL/$HARBOR_REPO:white .
          # Docker 이미지 빌드 및 태그 설정

          docker push $HARBOR_URL/$HARBOR_REPO:white
          # 빌드한 이미지를 Harbor 레지스트리에 Push
          '''
        }
      }
    }

    stage('Configure Nodes') {  // Ansible Playbook을 실행하여 노드 설정 및 Kubernetes 리소스 배포
      steps {
        sh '''
        ansible-playbook playbook.yml
        # Ansible Playbook 실행. Docker 설정 및 Kubernetes 리소스 배포 자동화
        '''
      }
    }
  }

  post {
    success {
      echo 'Deployment and service creation successful on master with 3 replicas!'
      // 모든 단계가 성공적으로 완료되었을 때 메시지 출력
    }
    failure {
      echo 'Deployment failed. Check the logs for details.'
      // 실패 시 메시지 출력
    }
  }
}
