pipeline {
  agent any
  environment {
    // change REGISTRY if your Nexus registry host/port differs
    REGISTRY = "hnexus.vyturr.one:5000"
    IMAGE    = "demo/trial-app"
    TAG      = "${env.BUILD_NUMBER ?: 'local'}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        // build image
        sh 'docker build -t ${REGISTRY}/${IMAGE}:${TAG} .'
        sh 'docker tag ${REGISTRY}/${IMAGE}:${TAG} ${REGISTRY}/${IMAGE}:latest'
      }
    }

    stage('Login to Nexus') {
      steps {
        // Jenkins credential with ID nexus-docker-creds must exist
        withCredentials([usernamePassword(credentialsId: 'nexus-docker-creds', usernameVariable: 'N_USER', passwordVariable: 'N_PASS')]) {
          sh 'echo "$N_PASS" | docker login ${REGISTRY} -u "$N_USER" --password-stdin'
        }
      }
    }

    stage('Push') {
      steps {
        sh 'docker push ${REGISTRY}/${IMAGE}:${TAG}'
        sh 'docker push ${REGISTRY}/${IMAGE}:latest'
      }
    }
  }

  post {
    always {
      sh 'docker logout ${REGISTRY} || true'
    }
  }
}


