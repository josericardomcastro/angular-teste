/**
 *  Create (if needed) the kubernetes namespace
 */
def createNamespace (namespace) {
    echo "Creating namespace ${namespace} if needed"
    sh "[ ! -z \"\$(kubectl get ns ${namespace} -o name 2>/dev/null)\" ] || kubectl create ns ${namespace}"
}
/**
 * Deploy a chart into repo
 */
def buildChart (chartname, imagetag) {
  //git branch: 'master', credentialsId: 'jenkins-token', url: 'https://gitlab.io.bb.com.br/infra/helm-template.git'
  sh """
    git clone https://github.com/josericardomcastro/helm-template.git
    helm repo add stable http://charts.apps.io.bb.com.br
    helm repo update
    mv helm-template ${chartname}
    mv k8s-values.yaml ${chartname}/values.yaml
    sed -i 's|image-name-ci|${imagetag}|g' ${chartname}/values.yaml
    mv Chart.yaml ${chartname}/Chart.yaml
  """
}

/**
 * Deploy a chart into kubernetes
 */
def deploymentChart (chartname) {
    echo "Deployment chart ${chartname} if needed"
    sh """
        if [ \"\$(helm ls | grep ${chartname} | wc -l)\" -eq 0 ];
        then
          helm install --name=${chartname} \
                       --namespace=${chartname} \
                       --set fullnameOverride=${chartname} \
                       ${chartname}/;
        else
          helm search
          helm repo update
          helm search
          helm upgrade ${chartname} \
                        --namespace=${chartname} \
                        ${chartname}/;
        fi
      """
}

/**
 * Build a docker image and push into repository
 */
def buildImageDocker(image, imagetag){
  echo "Build image: ${imagetag}"
  sh """
    docker build . -t ${image}
    docker tag ${image} ${imagetag}
    docker push ${imagetag}
  """
}


pipeline {

  options {
    // Build auto timeout
    timeout(time: 60, unit: 'MINUTES')
  }

  environment {
    PROJECT_NAME = "angular-teste"
    VERSION = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
    IMAGE = "josericardomcastro/${PROJECT_NAME}"
    IMAGE_TAG = "${IMAGE}:${VERSION}"
    NAMESPACE = "angular-teste"
    DEPLOYMENT_STATUS = ""
  }

  agent {
    kubernetes {
      label 'angular-teste'
      defaultContainer 'angular-cli'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: ci
spec:
  serviceAccountName: jenkins-service-account
  containers:
  - name: angular-cli
    image: teracy/angular-cli:1.5.0
    command:
    - cat
    tty: true
  - name: helm
    image: registry.io.bb.com.br:3389/helm
    command:
    - cat
    tty: true
  - name: docker
    image: docker:18.06.0-ce
    command:
    - cat
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: "/var/run/docker.sock"
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
    }
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Check out the code"
        //checkout scm
        echo "version => ${VERSION}"
      }
    }

    stage('Install dependencies') {
      steps {
        container('angular-cli') {
          echo "Install npm dependencies"
          sh 'npm config set registry https://nexus.apps.io.bb.com.br/repository/npm-group/'
          sh 'npm install'
        }
      }
    }

    stage('Test') {
      steps {
        echo 'Testing the code'
      }
    }

    stage('Build code') {
      steps {
        container('angular-cli') {
          echo "Build code in production version"
          sh 'ng build --prod'
        }
      }
    }

    stage('Namespace k8s') {
      steps {
        container('helm') {
          echo "Creating or updating the namespace ${PROJECT_NAME}"
          createNamespace ("${PROJECT_NAME}")
        }
      }
    }

    stage('Build image') {

      steps {
        container('docker') {
          buildImageDocker("${IMAGE}", "${IMAGE_TAG}")
        }
      }
    }

    stage('Build chart') {
      steps {
        echo "Build chart: ${PROJECT_NAME}"
        container('helm') {
          buildChart ("${PROJECT_NAME}", "${IMAGE_TAG}")
        }
      }
    }

    stage('Deploy chart') {

      steps {
        echo "Deploy chart: ${PROJECT_NAME}"
        container('helm') {
          deploymentChart ("${PROJECT_NAME}")
        }
      }
    }

  }
}
