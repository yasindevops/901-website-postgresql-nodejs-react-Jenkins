pipeline {
    agent any

    environment {
        DOCKERHUB_USER="yasindevops06"
        APP_REPO_NAME = "todo-app"   
        DB_VOLUME="myvolume"    
        NETWORK="mynetwork"             
    }

    stages {
        stage('Build App Docker Image') {
            steps {
                echo 'Building App Image'                
                sh 'docker build --force-rm -t "$DOCKERHUB_USER/$APP_REPO_NAME:postgre" -f ./database/Dockerfile .'
                sh 'docker build --force-rm -t "$DOCKERHUB_USER/$APP_REPO_NAME:nodejs" -f ./server/Dockerfile .'
                sh 'docker build --force-rm -t "$DOCKERHUB_USER/$APP_REPO_NAME:react" -f ./client/Dockerfile .'
                sh 'docker image ls'
            }
       }

        stage('Push Image to Dockerhub Repo') {
            steps {
                echo 'Pushing App Image to DockerHub Repo'
                withCredentials([string(credentialsId: 'my-dockerhub-token', variable: 'DOCKERHUB_TOKEN')]) {
                sh 'docker login -u yasindevops06 -p ${DOCKERHUB_TOKEN}'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:postgre"'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:nodejs"'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:react"'
            }
          }
        }
        stage('Create Volume') {
            steps {
                echo 'Creating  the volume for app and db containers.'
                sh 'docker volume create $DB_VOLUME' 
            }
        }
        stage('Create Network') {
            steps {
                echo 'Creating the network  for app and db containers.'
                sh 'docker network create $NETWORK' 
            }
        }

        stage('Deploy the DB') {
            steps {
                echo 'Deploying the DB'
                withCredentials([string(credentialsId: 'project901-postgre-password', variable: 'POSTGRE_PASS')]) {
                    sh 'docker run --name db -p 5432:5432 -v $DB_VOLUME:/var/lib/postgresql/data --network $NETWORK -e POSTGRES_PASSWORD=${POSTGRES_PASS} --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:postgre' 
            }
          }  
        }

        stage('wait the DB-container') {
            steps {
                script {
                    echo 'Waiting for the DB'
                    sh 'sleep  60s' // wait for  1 minute
                }
            }
        }

        stage('Deploy the server') {
            steps {
                echo 'Deploying the server'
                sh 'docker run --name server -p 5000:5000 --network $NETWORK --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:nodejs' 
            }
        }

        stage('wait the Nodejs-container') {
            steps {
                script {
                    echo 'Waiting for the Nodejs'
                    sh 'sleep  30s' // wait for  30 seconds
                }
            }
        }

        stage('Deploy the client') {
            steps {
                echo 'Deploying the client'
                sh 'docker run --name client -p 3000:3000 --network $NETWORK  --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:react' 
            }
        }

         stage('Destroy the infrastructure') {
            steps {
                timeout(time:5, unit:'DAYS') {
                    input message:'Approve terminate'
                }
                sh 'docker rm -f $(docker ps -aq)' 
                sh 'docker network rm $NETWORK'
                sh 'docker volume rm $DB_VOLUME'
                sh 'docker rmi -f $DOCKERHUB_USER/$APP_REPO_NAME:postgre $DOCKERHUB_USER/$APP_REPO_NAME:nodejs $DOCKERHUB_USER/$APP_REPO_NAME:react ' 
            }
        }
    }

    post {
        success {
        script {
        slackSend channel: '#class-chat', color: '#439FE0', message: ':fire: :fire:', teamDomain: 'devops15tr', tokenCredentialId: 'jenkins-slack'
            }
    }
    }  

}
