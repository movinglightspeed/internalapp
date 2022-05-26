pipeline {
    agent any 
    environment {
        registryCredential = 'dockerhub'
        imageName = 'stanleys/internalapp'
        dockerImage = ''
        }
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieve source from github. run npm install and npm test' 
                git branch: 'master',
                    url: 'https://github.com/movinglightspeed/internalapp.git'
                echo 'repo files'
                sh 'ls -a'
                echo 'install dependencies'
                sh 'npm install'
                echo 'Run tests'
                sh 'npm test'
                echo 'Testing completed'
            }
        }
        stage('Building image') {
            steps{
                script {
                    echo 'build the image' 
                    dockerImage = docker.build("${env.imageName}:${env.BUILD_ID}")
                    echo 'image built'
                }
            }
            }
        stage('Push Image') {
            steps{
                script {
                    echo 'push the image to docker hub' 
                    docker.withRegistry('',registryCredential){
                        dockerImage.push("${env.BUILD_ID}")
                    }
                }
            }
        }     
        stage('deploy to k8s') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                        }
                    }
            steps {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project roidtc-may2022-u306'
                sh "kubectl set image deployment/events-internal events-internal=${env.imageName}:${env.BUILD_ID}"
            }
        }     
        stage('Remove local docker image') {
            steps{
                echo "pending"
                //sh "docker rmi $imageName:latest"
                //sh "docker rmi $imageName:$BUILD_NUMBER"
                sh "docker rmi ${env.imageName}:${env.BUILD_ID}"
            }
        }
    }
}
