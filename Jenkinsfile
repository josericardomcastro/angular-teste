/*
    Create the kubernetes namespace
 */
def createNamespace (namespace) {
    echo "Creating namespace ${namespace} if needed"
    sh "[ ! -z \"\$(kubectl get ns ${namespace} -o name 2>/dev/null)\" ] || kubectl create ns ${namespace}"
}



pipeline {

  options {
    // Build auto timeout
    timeout(time: 60, unit: 'MINUTES')
  }

  environment {
    VERSION = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
    IMAGE = "josericardomcastro/${JOB_NAME}"
  }

  agent {
    kubernetes {
      label 'angular-cli'
      containerTemplates {
        containerTemplate {
          name 'docker'
          image 'docker:18.06.0-ce'
          ttyEnabled true
          command 'cat'
        }

        containerTemplate {
          name 'angular-cli'
          image 'teracy/angular-cli:1.5.0'
          ttyEnabled true
          command 'cat'
        }
      }
    }
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Check out angular code"
        checkout scm
        sh "echo ${VERSION}"
        sh 'ls -l'
        echo "version => ${VERSION}"
      }
    }

    stage('Install dependencies') {
      steps {
        echo "Install npm dependencies"
        sh 'npm install'
        sh 'ls -l'
      }
    }

    stage('Test') {
      steps {
        echo 'Testing the code'
      }
    }

    stage('Build code') {
      steps {
        echo "Build code in production version"
      }
    }

    stage('Build image') {

      agent {
        kubernetes {
          label 'docker'
          containerTemplate {
            name 'docker'
            image 'docker:18.06.0-ce'
            ttyEnabled true
            command 'cat'
          }
        }
      }

      steps {
        echo "Build image"
        sh """
          docker build -t ${IMAGE} .
          docker tag ${IMAGE} ${IMAGE}:${VERSION}
          docker push ${IMAGE}:${VERSION}
        """
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploying in kubernetes'
      }
    }
  }

}
