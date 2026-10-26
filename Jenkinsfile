pipeline {
  agent { label 'podman' }
  triggers { pollSCM('H/2 * * * *') }
  environment {
    REGISTRY = 'registry.assign.habdosh1.300.ops:5000'
    IMAGE    = "${REGISTRY}/bind-secondary"
  }
  stages {
    stage('Build') {
      steps { 
        sh 'podman build -t $IMAGE:$BUILD_NUMBER .' 
      }
    }
    stage('CVE Scan') {
      steps {
        sh 'rm -f image.tar'
        sh 'podman save -o image.tar $IMAGE:$BUILD_NUMBER'
        // Collect results (exit-code 0) but do not fail the build
        sh 'trivy image --input image.tar --severity HIGH,CRITICAL --exit-code 0'
      }
    }
    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'registry-login',
                         usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
          sh 'echo "$REG_PASS" | podman login -u "$REG_USER" --password-stdin $REGISTRY'
          sh 'podman push $IMAGE:$BUILD_NUMBER'
          sh 'podman logout $REGISTRY'
        }
      }
    }
  }
}
