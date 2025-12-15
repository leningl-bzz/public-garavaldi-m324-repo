pipeline {
  agent any
  tools { nodejs "node" }

  environment {
    // your-docker-hub-account/traffic-api
    IMAGE = "YOUR_DOCKERHUB_USER/traffic-api"
    BUILD_TAG = "v1.0.${BUILD_NUMBER}"

    // name of running container
    CONTAINER_NAME = "traffic-api-test"
  }

  stages {

    stage('Cloning Repo') {
      steps {
        script {
          checkout scm
        }
      }
    }

    stage('Building image') {
      steps {
        script {
          sh """
            set -e
            docker build -t ${IMAGE}:${BUILD_TAG} .
          """
        }
      }
    }

    stage('Install jest-cli') {
      steps {
        script {
          sh """
            # disable break-on-error
            set +e

            # check if jest-cli is installed
            JEST_INSTALLED=\$(npm ls -g -p | grep jest-cli)

            if [ -z "\$JEST_INSTALLED" ]; then
              echo "jest-cli not found, installing..."
              npm install -g jest-cli

              if [ \$? -ne 0 ]; then
                echo "ERROR: jest-cli installation failed"
                exit 1
              fi
            else
              echo "jest-cli already installed"
            fi

            # enable break-on-error
            set -e
          """
        }
      }
    }

    stage('Install dependencies') {
      steps {
        sh """
          set -e
          npm install
        """
      }
    }

    stage('Run container for testing') {
      steps {
        sh """
          set -e
          docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE}:${BUILD_TAG}
        """
      }
    }

    stage('Test') {
      steps {
        sh """
          set -e
          npm test
        """
      }
    }

    stage('Remove traffic container') {
      steps {
        sh """
          set +e
          docker rm -f ${CONTAINER_NAME}
          set -e
        """
      }
    }

    stage('Remove Unused docker image') {
      steps {
        sh """
          set +e
          docker rmi -f ${IMAGE}:${BUILD_TAG}
          docker image prune -af
          set -e
        """
      }
    }
  }
}
