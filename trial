mkdir -p ~/hello-app && cd ~/hello-app

# app
mkdir -p app
cat > app/main.py <<'PY'
print("Hello from CI build!")
PY

# Dockerfile
cat > Dockerfile <<'DOCK'
FROM python:3.12-slim
WORKDIR /app
COPY app/ /app/
CMD ["python","main.py"]
DOCK

# Jenkinsfile (Pipeline)
cat > Jenkinsfile <<'JENK'
pipeline {
  agent any
  environment {
    REGISTRY = "hnexus.vyturr.one:5000"    // change to "hnexus.vyturr.one" if you proxy registry over 443
    IMAGE    = "demo/hello-app"
    TAG      = "${env.BUILD_NUMBER}"
  }
  stages {
    stage("Checkout") { steps { checkout scm } }
    stage("Docker Build") {
      steps {
        sh '''
          docker build -t ${REGISTRY}/${IMAGE}:${TAG} .
          docker tag ${REGISTRY}/${IMAGE}:${TAG} ${REGISTRY}/${IMAGE}:latest
        '''
      }
    }
    stage("Login to Nexus") {
      steps {
        withCredentials([usernamePassword(credentialsId: "nexus-docker-creds", usernameVariable: "U", passwordVariable: "P")]) {
          sh 'echo "$P" | docker login ${REGISTRY} -u "$U" --password-stdin'
        }
      }
    }
    stage("Push Images") {
      steps {
        sh '''
          docker push ${REGISTRY}/${IMAGE}:${TAG}
          docker push ${REGISTRY}/${IMAGE}:latest
        '''
      }
    }
  }
  post { always { sh 'docker logout ${REGISTRY} || true' } }
}
JENK

