pipeline {
  agent any
  environment {
    NEXUS_BASE = "http://hnexus.vyturr.one:8081"   // change to your Nexus base URL if needed
    NEXUS_REPO = "raw-hosted"                     // raw hosted repo name
    NEXUS_UPLOAD_PATH = "ci-artifacts"            // base path inside repo
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
        withCredentials([usernamePassword(credentialsId: 'nexus-http-creds', usernameVariable: 'N_USER', passwordVariable: 'N_PASS')]) {
          script {
            // compute remote folder per build number and timestamp
            def remoteFolder = "${env.NEXUS_UPLOAD_PATH}/${env.BUILD_NUMBER}"
            echo "Uploading to Nexus path: ${remoteFolder}"

            // upload each file in build/
            sh """
              set -e
              for f in build/*; do
                fname=\$(basename \"\$f\")
                url="${NEXUS_BASE}/repository/${NEXUS_REPO}/${remoteFolder}/\$fname"
                echo "Uploading \$f => \$url"
                # use curl with basic auth; --fail to catch HTTP errors; --show-error for diagnostics
                curl -u "$N_USER:$N_PASS" --fail --show-error --upload-file "\$f" "\$url"
                echo "Uploaded: \$url"
              done
            """
            
            // expose the base URL as an environment variable for later use
            env.NEXUS_ARTIFACT_URL = "${NEXUS_BASE}/repository/${NEXUS_REPO}/${remoteFolder}/"
            echo "Artifacts uploaded to: ${env.NEXUS_ARTIFACT_URL}"
          }
        }
      }
    }
  }

  post {
    success {
      echo "Done. Nexus artifact folder: ${env.NEXUS_ARTIFACT_URL}"
    }
    failure {
      echo "Upload failed — check console output"
    }
  }
}


