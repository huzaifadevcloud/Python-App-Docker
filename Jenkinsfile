pipeline {
  agent { label "ec2"
  }

  environment {
    IMAGE_NAME = 'huzaifafev/python-docker-image'
    IMAGE_TAG = 'latest'
    FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"

    DOCKER_CREDENTIALS_ID = 'dockerhub-creds'   // Jenkins credentials for Docker Hub
  }

  stages {

    stage('Checkout Code') {
      steps {
        echo 'Checking out code from the github'
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        echo 'Building Docker image...'
        sh "docker build -t ${FULL_IMAGE} ."
      }
    }

    stage('Push Docker Image') {
      steps {
        echo 'Pushing Docker image to Docker Hub...'
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${FULL_IMAGE}
            docker logout
          '''
        }
      }
    }

    stage('Run App Container') {
      steps {
        echo 'Running Docker container...'
        //Run webapp independently period
        sh "docker run --rm -p 8000:8000 ${FULL_IMAGE}"
        // Run the web app and test it
        /*sh '''
        docker run -d --name test_container -p 8000:8000 ${FULL_IMAGE}
        sleep 5  
        curl -f http://localhost:8000/demo || echo "Server did not respond"
        docker stop test_container
        docker rm test_container
        '''*/
      }
    }

  }

  post {
    always {
      echo 'Cleaning up...'
      sh 'docker image prune -f'
    }
  }
}
