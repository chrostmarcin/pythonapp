pipeline {
    
    agent any
    

    environment {
        dockerImage =''
        registry = 'chrostmarcin/nginx'
        registryCredential ='docker_id'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/chrostmarcin/pythonapp.git']])
            }
        }
        
        stage('Docker image') {
            steps {
                script {
                    dockerImage = docker.build registry
                                  
                     }
            }
        }
        
        stage ('Uploading Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential ) {
                        sh 'docker tag chrostmarcin/nginx chrostmarcin/nginx:1.2'
                        sh 'docker push chrostmarcin/nginx:1.2'
                    }
                }    
            }
        }
    }
}
