pipeline {

//   agent none
  agent any

  environment {
    DOCKER_IMAGE = "sangkiemtoan/flask-docker"
  }

  stages {
    stage("Test") {
      agent {
          docker {
            image 'python:3.8-slim-buster'
            args '-u 0:0 -v /tmp:/root/.cache'
          }
      }
      steps {
        sh "pip install poetry"
        sh "poetry install"
        sh "poetry run pytest"
      }
    }

    stage("build") {
      agent { node {label 'master'}}
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
        sh "docker image ls | grep ${DOCKER_IMAGE}"
        withCredentials([usernamePassword(credentialsId: 'docker.hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
            sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
            sh "docker push ${DOCKER_IMAGE}:latest"
        }

        // //clean to save disk
        // sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
        // sh "docker image rm ${DOCKER_IMAGE}:latest"
      }
    }
    // stage("deploy") { 
    //     steps {
    //         // withCredentials([sshKey(credentialsId: 'ssh-key', , sshKeyVariable: 'SSH_KEY')]) {
    //         //     sh "ssh -i $SSH_KEY jenkins@159.65.142.215 './deploy.sh'"
    //         // }

    //         // sshagent(['']) {
    //         //     sh "ssh -i jenkins@159.65.142.215 './deploy.sh'"
    //         // def runDeploy = './deploy.sh'
    //         sshagent(['c8ce0eda-46b0-4dde-bf68-473999123696']) {
    //             sh "ssh -o StrictHostKeyChecking=no jenkins@188.166.233.251 './deploy.sh'"
    //         }
    //     }
    // }
}  
  

  post {
    success {
      echo "SUCCESSFUL"
    }
    failure {
      echo "FAILED"
    }
  }
}
