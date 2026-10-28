pipeline {

  agent any

  stages { 

    stage ('Checkout') {
        steps {
            git branch: 'main',
                url: 'https://github.com/aishwaryan153/proj-3-aws-jenkins-k8s.git'
              }
    }

    stage ('Build docker image') {
        steps {
            sh 'docker build -t proj3:latest .'
        }
    }

    stage ('tag docker image') {
       steps {
            sh 'docker tag proj3-repo:latest aishwaryalaxmi0599/proj3:latest'
       }
    }

    stage ('login to docker hub') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USERNAME'
                    passwordVariable: 'DOCKER_PASSWORD'
                  )
              ]) {
                  sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
            }
        }
    }

    stage ('Push docker image') {
        steps {
            sh 'docker push aishwaryalaxmi0599/proj3:latest'
        }
    }
  }

}
