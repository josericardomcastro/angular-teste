/*
    Create the kubernetes namespace
 */
def createNamespace (namespace) {
    echo "Creating namespace ${namespace} if needed"
    sh "[ ! -z \"\$(kubectl get ns ${namespace} -o name 2>/dev/null)\" ] || kubectl create ns ${namespace}"
}

def commitId

pipeline {

  options {
    // Build auto timeout
    timeout(time: 60, unit: 'MINUTES')
  }

  agent {
    kubernetes {
      label 'angular-cli'
      containerTemplate {
        name 'angular-cli'
        image 'teracy/angular-cli:1.5.0'
        ttyEnabled true
        command 'cat'
      }
    }
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Check out angular code"
        checkout scm
        commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        sh 'ls -l'
        echo "commitid => ${commitId}"
      }
    }

    stage('Install dependencies') {
      steps {
        echo "Install npm dependencies"
        sh 'npm install'
        sh 'ls -l'
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
