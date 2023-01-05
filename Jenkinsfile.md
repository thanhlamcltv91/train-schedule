pipeline {
    agent {
      kubernetes  {
            label 'jenkins-slave'
             defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker:18.09
    command:
    - cat
    tty: true
  - name: tools
    image: tuannanhh/train-schedule:latest
    command:
    - cat
    tty: true
"""
        }
    }
  environment {
      IMAGE_REPO = "<DOCKERHUB_USER>/train-schedule"
      // Instead of DOCKERHUB_USER, use your Dockerhub name
  }
  stages {
    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub_tuananh')
      }
      steps {
        container('docker') {
          sh "echo ${env.GIT_COMMIT}"
          // Build new image
          sh "until docker container ls; do sleep 3; done && docker image build -t  ${env.IMAGE_REPO}:${env.GIT_COMMIT} ."
          // Publish new image
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker image push ${env.IMAGE_REPO}:${env.GIT_COMMIT}"
        }
      }
    }
    stage('Deploy') {
      environment {
        GIT_CREDS = credentials('tuananh_github')
        HELM_GIT_REPO_URL = "github.com/<GITHUB_USERNAME>/rsvpapp-helm-cicd.git"
        GIT_REPO_EMAIL = '<PUT_EMAIL_ID>'
        GIT_REPO_BRANCH = "main"
          
       // Update above variables with your user details
      }
      steps {
        container('tools') {
            sh "git clone https://${env.HELM_GIT_REPO_URL}"
            sh "git config --global user.email ${env.GIT_REPO_EMAIL}"
             // install wq
            sh "wget https://github.com/mikefarah/yq/releases/download/v4.9.6/yq_linux_amd64.tar.gz"
            sh "tar xvf yq_linux_amd64.tar.gz"
            sh "mv yq_linux_amd64 /usr/bin/yq"
            sh "git checkout -b master"
          dir("rsvpapp-helm-cicd") {
              sh "git checkout ${env.GIT_REPO_BRANCH}"
            //install done
            sh '''#!/bin/bash
              echo $GIT_REPO_EMAIL
              echo $GIT_COMMIT
              ls -lth
              yq eval '.image.repository = env(IMAGE_REPO)' -i values.yaml
              yq eval '.image.tag = env(GIT_COMMIT)' -i values.yaml
              cat values.yaml
              pwd
              git add values.yaml
              git commit -m 'Triggered Build'
              git push https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/$GIT_CREDS_USR/rsvpapp-helm-cicd.git
            '''
          }
        }
      }
    }   
  }
}