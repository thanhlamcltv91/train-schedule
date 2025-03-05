pipeline {
    agent any
 
    stages {
        stage('check out sourcecode') {
            steps {
            deleteDir()    
            checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/thanhlamcltv91/train-schedule.git']])
            }
        }
        stage('build images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker.com', passwordVariable: 'f1&kdLnnNSzsYhXb', usernameVariable: 'thanhlamcltv91')]) {
                    sh 'sudo -S docker build -t tel4vn:v${BUILD_NUMBER} .'
                    sh 'cat server.js'
                }
            }
        }
        
        stage('push images to repo') {
            steps {
                sh 'sudo docker tag tel4vn:v${BUILD_NUMBER} thanhlamcltv91/tel4vn:v${BUILD_NUMBER}'
                sh 'sudo docker login -u thanhlamcltv91 -p f1&kdLnnNSzsYhXb'
                sh 'sudo docker push thanhlamcltv91/tel4vn:v${BUILD_NUMBER}'
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
                build job: 'tel4vn-update-for argocd', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
        }
    }
}
