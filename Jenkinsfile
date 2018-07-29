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

  agent {
    kubernetes {
      label 'nginx'
      containerTemplate {
        name 'nginx'
        image 'nginx:1.15.2'
        ttyEnabled true
        command 'cat'
      }
    }
  }

  stages {
    stage('Checkout') {

      agent {
        label 'angular-cli'
      }

      steps {
        echo "Check out angular code"
        checkout scm
        sh 'ls -l'
        sh 'npm install'
      }
    }

    stage('Install dependencies') {

      agent {
        label 'nginx'
      }

      steps {
        echo "Install npm dependencies"
      }
    }

    stage('Build') {

      agent {
        label 'nginx'
      }

      steps {
        echo "Build code in production version"
      }
    }

    stage('Test') {

      agent {
        label 'nginx'
      }

      steps {
        echo 'Testing the code'
      }
    }

    stage('Deploy') {

      agent {
        label 'nginx'
      }

      steps {
        echo 'Deploying in kubernetes'
      }
    }
  }

}
