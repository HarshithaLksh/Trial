pipeline {
  agent any
  environment {
    NEXUS_BASE = "http://hnexus.vyturr.one:8081"   // adjust if needed
    NEXUS_REPO  = "raw-hosted"
    REMOTE_BASE = "ci-artifacts"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare artifacts') {
      steps {
        sh '''
          mkdir -p build
          echo "Hello from CI build!" > build/hello.txt
          echo "ok" > build/test-result.txt
          ls -l build
        '''
      }
    }

    stage('Archive locally') {
      steps {
        archiveArtifacts artifacts: 'build/**', fingerprint: true
      }
    }

    stage('Upload to Nexus (raw)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'nexus-http-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PW')]) {
          sh '''
            set -euo pipefail
            REMOTE_FOLDER="${REMOTE_BASE}/${BUILD_NUMBER}"
            for f in build/*; do
              fname=$(basename "$f")
              url="${NEXUS_BASE}/repository/${NEXUS_REPO}/${REMOTE_FOLDER}/${fname}"
              echo "Uploading $f => $url"
              curl -u "$NEXUS_USER:$NEXUS_PW" --fail --show-error --upload-file "$f" "$url"
              echo "Uploaded: $url"
            done
            echo "Uploaded artifacts to: ${NEXUS_BASE}/repository/${NEXUS_REPO}/${REMOTE_FOLDER}/"
          '''
        }
      }
    }
  }
  post {
    success {
      echo "Pipeline complete. Artifacts uploaded."
    }
    failure {
      echo "Pipeline failed; check console output."
    }
  }
}


