pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare') {
      steps {
        sh '''
          echo "Workspace: $WORKSPACE"
          mkdir -p build
          echo "Hello hi from simple pipeline" > build/hello.txt
        '''
      }
    }

    stage('Print') {
      steps {
        sh 'echo "Contents of build/hello.txt:" && cat build/hello.txt'
      }
    }

    stage('Unit Test (dummy)') {
      steps {
        sh '''
          echo "Running a dummy test..."
          echo "ok" > build/test-result.txt
          test -f build/test-result.txt && echo "Test passed"
        '''
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'build/**', fingerprint: true
      }
    }
  }

  post {
    success { echo "Pipeline succeeded." }
    failure { echo "Pipeline failed — check console output." }
    always { cleanWs() }
  }
}

