pipeline {
  agent any

  environment {
    APP_NAME = "trial-app"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare') {
      steps {
        // create a virtual env and install dependencies if any (optional)
        sh '''
          echo "Workspace: $WORKSPACE"
          mkdir -p build
          echo "print('Hello from simple pipeline')" > app.py
        '''
      }
    }

    stage('Run Script') {
      steps {
        // run the Python script (assumes python is available on the agent)
        sh '''
          python --version || true
          python app.py
        '''
      }
    }

    stage('Unit Test') {
      steps {
        // a tiny "test" (creates a file then validates it)
        sh '''
          echo "Running a simple test..."
          echo "ok" > build/test-result.txt
          test -f build/test-result.txt && echo "Test file exists"
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
    success {
      echo "Pipeline succeeded."
    }
    failure {
      echo "Pipeline failed — check console output."
    }
    always {
      cleanWs()
    }
  }
}

