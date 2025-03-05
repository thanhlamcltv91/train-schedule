pipeline {
    agent { label 'tel4vn' }
 
    stages {
        stage('check out sourcecode') {
            steps {
                deleteDir()
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'thanhlamcltv91', url: 'https://github.com/thanhlamcltv91/train-schedule.git']])
            }
        }
        stage('build images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'SUDO_PASSWORD', usernameVariable: 'SUDO_USERNAME')]) {
                    sh 'echo $SUDO_PASSWORD | sudo -S docker build -t tel4vn:v${BUILD_NUMBER} .'
                    sh 'cat server.js'
                }
            }
        }
        stage('push images to repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'sudo docker tag tel4vn:v${BUILD_NUMBER} thanhlamcltv91/tel4vn:v${BUILD_NUMBER}'
                    sh 'echo $DOCKER_PASSWORD | sudo -S docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'sudo docker push thanhlamcltv91/tel4vn:v${BUILD_NUMBER}'
                }
            }
        }
        stage('clean images local') {
            steps {
                sh 'sudo docker rmi tel4vn:v${BUILD_NUMBER}'
                sh 'sudo docker rmi thanhlamcltv91/tel4vn:v${BUILD_NUMBER}'
            }
        }
        stage('run update images for argocd') {
            steps {
                build job: 'tel4vn-update-for-argocd', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
        }
    }
}
