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

  agent none

  pod {
    containers {
      container {
        name 'angular-cli'
        image 'teracy/angular-cli:1.5.0'
      }
    }
  }

  stages {
    stage('Checkout') {
      agent {
        container {
          name 'angular-cli'
        }
      }
      steps {
        echo "Check out angular code"
        checkout scm
        sh 'ls -l'
        sh 'npm install'
      }
    }

    stage('Install dependencies') {
      steps {
        echo "Install npm dependencies"
      }
    }

    stage('Build') {
      steps {
        echo "Build code in production version"
      }
    }

    stage('Test') {
      steps {
        echo 'Testing the code'
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploying in kubernetes'
      }
    }
  }

}
