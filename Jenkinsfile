pipeline {
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
  }
  agent any
  environment {
    IMAGE            = "nginx-proxy-manager-base"
    TEMP_IMAGE       = "${IMAGE}-build_${BUILD_NUMBER}"
    // Architectures:
    AMD64_TAG        = "amd64"
    ARMV6_TAG        = "armv6l"
    ARMV7_TAG        = "armv7l"
    ARM64_TAG        = "arm64"
  }
  stages {
    stage('Build') {
      when {
        branch 'master'
      }
      parallel {
        // ========================
        // amd64
        // ========================
        stage('amd64') {
          agent {
            label 'amd64'
          }
          steps {
            // Docker Build
            sh 'docker build --pull --no-cache --squash --compress -t ${TEMP_IMAGE}-${AMD64_TAG} .'

            // Dockerhub
            sh 'docker tag ${TEMP_IMAGE}-${AMD64_TAG} docker.io/jc21/${IMAGE}:latest-${AMD64_TAG}'
            withCredentials([usernamePassword(credentialsId: 'jc21-dockerhub', passwordVariable: 'dpass', usernameVariable: 'duser')]) {
              sh "docker login -u '${duser}' -p '${dpass}'"
              sh 'docker push docker.io/jc21/${IMAGE}:latest-${AMD64_TAG}'
            }

            sh 'docker rmi ${TEMP_IMAGE}-${AMD64_TAG}'
          }
        }
        // ========================
        // arm64
        // ========================
        stage('arm64') {
          agent {
            label 'arm64'
          }
          steps {
            // Docker Build
            sh 'docker build --pull --no-cache --squash --compress -f Dockerfile.${ARM64_TAG} -t ${TEMP_IMAGE}-${ARM64_TAG} .'

            // Dockerhub
            sh 'docker tag ${TEMP_IMAGE}-${ARM64_TAG} docker.io/jc21/${IMAGE}:latest-${ARM64_TAG}'
            withCredentials([usernamePassword(credentialsId: 'jc21-dockerhub', passwordVariable: 'dpass', usernameVariable: 'duser')]) {
              sh "docker login -u '${duser}' -p '${dpass}'"
              sh 'docker push docker.io/jc21/${IMAGE}:latest-${ARM64_TAG}'
            }

            sh 'docker rmi ${TEMP_IMAGE}-${ARM64_TAG}'
          }
        }
        // ========================
        // armv7l
        // ========================
        stage('armv7l') {
          agent {
            label 'armv7l'
          }
          steps {
            // Docker Build
            sh 'docker build --pull --no-cache --squash --compress -f Dockerfile.${ARMV7_TAG} -t ${TEMP_IMAGE}-${ARMV7_TAG} .'

            // Dockerhub
            sh 'docker tag ${TEMP_IMAGE}-${ARMV7_TAG} docker.io/jc21/${IMAGE}:latest-${ARMV7_TAG}'
            withCredentials([usernamePassword(credentialsId: 'jc21-dockerhub', passwordVariable: 'dpass', usernameVariable: 'duser')]) {
              sh "docker login -u '${duser}' -p '${dpass}'"
              sh 'docker push docker.io/jc21/${IMAGE}:latest-${ARMV7_TAG}'
            }

            sh 'docker rmi ${TEMP_IMAGE}-${ARMV7_TAG}'
          }
        }
        // ========================
        // armv6l - Disabled for the time being
        // ========================
        /*
        stage('armv6l') {
          agent {
            label 'armv6l'
          }
          steps {
            // Docker Build
            sh 'docker build --pull --no-cache --squash --compress -f Dockerfile.armv6 -t ${TEMP_IMAGE}-${ARMV6_TAG} .'

            // Dockerhub
            sh 'docker tag ${TEMP_IMAGE}-${ARMV6_TAG} docker.io/jc21/${IMAGE}:latest-${ARMV6_TAG}'
            withCredentials([usernamePassword(credentialsId: 'jc21-dockerhub', passwordVariable: 'dpass', usernameVariable: 'duser')]) {
              sh "docker login -u '${duser}' -p '${dpass}'"
              sh 'docker push docker.io/jc21/${IMAGE}:latest-${ARMV6_TAG}'
            }

            sh 'docker rmi ${TEMP_IMAGE}-${ARMV6_TAG}'
          }
        }
        */
      }
    }
    // ========================
    // latest manifest
    // ========================
    stage('Latest Manifest') {
      when {
        branch 'master'
      }
      steps {
        ansiColor('xterm') {
          // =======================
          // latest
          // =======================
          sh 'docker pull jc21/${IMAGE}:latest-${AMD64_TAG}'
          sh 'docker pull jc21/${IMAGE}:latest-${ARM64_TAG}'
          sh 'docker pull jc21/${IMAGE}:latest-${ARMV7_TAG}'
          sh 'docker pull jc21/${IMAGE}:latest-${ARMV6_TAG}'

          sh 'docker manifest push --purge jc21/${IMAGE}:latest || :'
          sh 'docker manifest create jc21/${IMAGE}:latest jc21/${IMAGE}:latest-${AMD64_TAG} jc21/${IMAGE}:latest-${ARM64_TAG} jc21/${IMAGE}:latest-${ARMV7_TAG} jc21/${IMAGE}:latest-${ARMV6_TAG}'

          sh 'docker manifest annotate jc21/${IMAGE}:latest jc21/${IMAGE}:latest-${AMD64_TAG} --arch ${AMD64_TAG}'
          sh 'docker manifest annotate jc21/${IMAGE}:latest jc21/${IMAGE}:latest-${ARM64_TAG} --os linux --arch ${ARM64_TAG}'
          sh 'docker manifest annotate jc21/${IMAGE}:latest jc21/${IMAGE}:latest-${ARMV7_TAG} --os linux --arch arm --variant ${ARMV7_TAG}'
          sh 'docker manifest annotate jc21/${IMAGE}:latest jc21/${IMAGE}:latest-${ARMV6_TAG} --os linux --arch arm --variant ${ARMV6_TAG}'
          sh 'docker manifest push --purge jc21/${IMAGE}:latest'
        }
      }
    }
    // ========================
    // cleanup
    // ========================
    stage('Latest Cleanup') {
      when {
        branch 'master'
      }
      steps {
        ansiColor('xterm') {
          sh 'docker rmi jc21/${IMAGE}:latest || echo ""'
          sh 'docker rmi jc21/${IMAGE}:latest-${AMD64_TAG} || echo ""'
          sh 'docker rmi jc21/${IMAGE}:latest-${ARM64_TAG} || echo ""'
          sh 'docker rmi jc21/${IMAGE}:latest-${ARMV7_TAG} || echo ""'
          sh 'docker rmi jc21/${IMAGE}:latest-${ARMV6_TAG} || echo ""'
        }
      }
    }
  }
  triggers {
    bitbucketPush()
  }
  post {
    success {
      juxtapose event: 'success'
      sh 'figlet "SUCCESS"'
    }
    failure {
      juxtapose event: 'failure'
      sh 'figlet "FAILURE"'
    }
  }
}
