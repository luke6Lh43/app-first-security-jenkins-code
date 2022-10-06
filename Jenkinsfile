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

    // stage('secureCN') {
    //     agent any
    //     steps {
    //         secureCNVulnerabilityScanner(imageName: registry + ":$BUILD_NUMBER",
    //         secureCnAccessKey: 'ee00ddf5-1af7-4858-a9da-68a3c3fcf3a2', secureCnSecretKeyId: 'L8y+yTveUXFs92q6hJFiN2TwtCV3Wk0SfWOLsQwH6F0=',
    //         highestSeverityAllowed: 'HIGH', dockerRegistryPasswordId: registryCredential,
    //         highestSeverityAllowedDf: 'FATAL', url: "securecn.cisco.com", pushLocalImage: 'true')
    //     }
    // }

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