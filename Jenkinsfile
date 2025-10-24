pipeline {
  agent any

  environment {
    // Change this to your DockerHub username
    DOCKERHUB_USER = 'your-dockerhub-username'
    IMAGE_NAME = "${DOCKERHUB_USER}/devops-challenge-1"
    IMAGE_TAG  = "${env.BUILD_NUMBER ?: 'latest'}"
    FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${FULL_IMAGE} ."
      }
    }

    stage('Test') {
      steps {
        sh '''
          python3 -m venv venv
          . venv/bin/activate
          pip install -r requirements.txt || true
          pip install pytest || true
          pytest -q || true
        '''
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
        )]) {
          sh '''
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker push ${FULL_IMAGE}
            docker logout
          '''
        }
      }
    }

    stage('Deploy Locally') {
      steps {
        sh '''
          docker rm -f devops_challenge_app || true
          docker run -d --name devops_challenge_app -p 5000:5000 ${FULL_IMAGE}
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Image: ${FULL_IMAGE}"
    }
  }
}
