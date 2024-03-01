pipeline{
    agent any
    tools {
           dockerTool 'docker'            
    }

    stages { 
        stage('Deploy the App') {
            steps {
                echo 'Deploy the App'
                sh 'ls -l'
                sh 'docker --version'
                script {
                    dockerComposeBuild(
                        composeFile: 'docker-compose.yaml')
                }        
             }
        }

    

        stage('Destroy the infrastructure'){
            steps{
                timeout(time:5, unit:'DAYS'){
                    input message:'Approve terminate'
                }
                script {
                    dockerComposeDown(
                        composeFile: 'docker-compose.yaml')
                } 
            }
        }
    }

    post {
        success {
        script {
        slackSend channel: '#class-chat', color: '#439FE0', message: ':fire: Project-207 with jenkins :fire:', teamDomain: 'devops15tr', tokenCredentialId: 'jenkins-slack'
            }
    }
    }  

}
