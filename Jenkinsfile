pipeline {
    agent any
    stages {
        // stage('Build') {
        //     steps {
        //         echo 'Running build automation'
        //         sh './gradlew build --no-daemon'
        //         archiveArtifacts artifacts: 'dist/trainSchedule.zip'
        //     }
        // }
        // stage('Checkout Source') {
        //     steps {
        //          git 'https://github.com/tuananh281/cicd-pipeline-train-schedule-dockerdeploy.git'
        //     }
        // }

        stage('Build docker images') {
            steps {
                script {
                    DOCKER_IMAGE="gitops-demo"
                    app = docker.build("tuannanhh/${DOCKER_IMAGE}")
                    // app.inside {
                    //     sh 'echo $(curl localhost:8080)'
                    // }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    DOCKER_REGISTRY="registry.hub.docker.com"
                    DOCKER_NAME="tuannanhh"

                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }

                    sh "docker image rm ${DOCKER_NAME}/${DOCKER_IMAGE}:latest"
                    sh "docker image rm ${DOCKER_REGISTRY}/${DOCKER_NAME}/${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Trigger ManifestUpdate') {
            steps {
                echo "Update manifestjob"
                build job: 'update-manifest-github', //name job 2
                    parameters: [
                        string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)
                    ]   
            }
        }
    
    }
}
//         stage('Deploy k8s') {
//             steps{
//                  withCredentials([string(credentialsId: "argocd_role", variable: 'ARGOCD_AUTH_TOKEN')]) {
//                      sh '''
//                      ARGOCD_SERVER="argocd-deploy.com"
//                      APP_NAME=tuananh-dep-k8s
                     
//                      ARGOCD_SERVER=$ARGOCD_SERVER argocd --grpc-web app sync $APP_NAME --force
//                      ARGOCD_SERVER=$ARGOCD_SERVER argocd --grpc-web app wait $APP_NAME --timeout 600
//                      '''
//                 }
//             }
//         }    
        
        // stage('Test') {
        //     steps {
        //         echo 'Testing...'
        //         sh 'npm test'
        //     }
        // }

//         stage('Deploying App to Kubernetes') {
//             steps {
//                 // input 'Do you want to deploy to K8S Cluster ?'
//                 // milestone(1)
//                 script {
//                     kubernetesDeploy(configs: 'deploy/deployment.yml', kubeconfigId: 'k8s_tuananh')
//                 }
//             }
//         }

//         stage('Deploy Docker To Development') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'webserver-deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
//                     script {
//                         sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$dev_ip \' docker pull tuannanhh/train-schedule:${env.BUILD_NUMBER}\'"
//                         try {
//                             sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$dev_ip \' docker stop train-schedule\'"
//                             sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$dev_ip \' docker rm train-schedule\'"
//                         } catch (err) {
//                             echo 'caucht error: $err'
//                         }
//                         sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$dev_ip \' docker run --restart always --name train-schedule -p 8080:8080 -d tuannanhh/train-schedule:${env.BUILD_NUMBER}\'"
//                     }
//                 }
//             }
//         }

//         stage('Deploy Docker To Stagging') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'webserver-deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
//                     script {
//                         sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$staging_ip \' docker pull tuannanhh/train-schedule:${env.BUILD_NUMBER}\'"
//                         try {
//                             sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$staging_ip \' docker stop train-schedule\'"
//                             sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$staging_ip \' docker rm train-schedule\'"
//                         } catch (err) {
//                             echo 'caucht error: $err'
//                         }
//                         sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$staging_ip \' docker run --restart always --name train-schedule -p 8080:8080 -d tuannanhh/train-schedule:${env.BUILD_NUMBER}\'"
//                     }
//                 }
//             }
//         }

//         stage('Deploy Docker To Production') {
//             steps {
//                 input 'Do you want to deploy to production ?'
//                 milestone(1)
//                 withCredentials([usernamePassword(credentialsId: 'webserver-deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
//                     script {
//                         sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$prod_ip \' docker pull tuannanhh/train-schedule:${env.BUILD_NUMBER}\'"
//                         try {
//                             sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$prod_ip \' docker stop train-schedule\'"
//                             sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$prod_ip \' docker rm train-schedule\'"
//                         } catch (err) {
//                             echo 'caucht error: $err'
//                         }
//                         sh "sshpass -p '$USERPASS' ssh -o 'StrictHostKeyChecking=no' $USERNAME@$prod_ip\' docker run --restart always --name train-schedule -p 8080:8080 -d tuannanhh/train-schedule:${env.BUILD_NUMBER}\'"
//                     }
//                 }
//             }
//         }
