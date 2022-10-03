pipeline {
  agent none
  stages {
    stage('Build') {
      agent {
        docker {
          image 'python:alpine3.7'
          args '-p 5000:5000'
        }

      }
      steps {
        sh 'pip install -r requirements.txt'
        sh 'apk add libstdc++'
        sh 'python ./app.py &'
      }
    }

    stage('Test App') {
      agent {
        docker {
          image 'python:alpine3.7'
          args '-p 5000:5000'
        }

      }
      post {
        always {
          junit 'test-reports/*.xml'
        }

      }
      steps {
        sh 'pip install -r requirements.txt'
        sh 'apk add libstdc++'
        sh 'python ./app.py &'
        echo "${env.NODE_NAME}"
        sh 'pwd'
        sh 'uname -a'
        sh 'python test.py'
      }
    }

    // stage('SAS Test') {
    //   agent { 
    //     docker { 
    //       image 'snyk/snyk-cli:python-3'
    //       args '-p 5000:5000'
    //         }
    //   }
    //   steps {
    //     sh 'pip install -r requirements.txt'
    //     sh 'apk add libstdc++'
    //     sh 'python ./app.py &'        
    //     snykSecurity(
    //       snykInstallation: 'SnykV2Plugin',
    //       snykTokenId: 'snyktoken',
    //       severity: 'medium',
    //       failOnIssues: true)
    //   }
    // }

    stage('Build image') {
      agent any
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }

      }
    }

    stage('Upload Image to Registry') {
      agent any
      steps {
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }

      }
    }

    stage ('Panoptica') {
        steps {
            dockerImage = docker.build registry + ":$BUILD_NUMBER"
            secureCNVulnerabilityScanner(imageName: dockerImage,
            secureCnAccessKey: 'a8d4aeb2-0812-40d4-be5a-69110a7c78ed', secureCnSecretKeyId: 'wZLOQfXr1Mw2FGe0S5P60h23IaDNMdTgievExIezLqo=',
            highestSeverityAllowed: 'HIGH', dockerRegistryPasswordId: registryCredential,
            highestSeverityAllowedDf: 'FAIAL', url: "appsecurity.cisco.com", pushLocalImage: 'true')
        }
    }

    stage('Remove Unused docker image') {
      agent any
      steps {
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }

  }
  environment {
    registry = 'lciukaj/my-cicd-app'
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
}