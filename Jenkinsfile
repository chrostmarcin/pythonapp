pipeline {
    
    agent any
    
    triggers {
        pollSCM '*/5 * * * *'
    }

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

        stage ('Testing Container') {
            steps {
                script {
                    sh 'docker run  -it --rm -d --name nginx -p 8181:80 chrostmarcin/nginx'
                    sh 'curl 192.168.68.120:8181'
                    sh 'docker rm -f nginx'
                    }
                }    
            }
        
        
        stage ('Uploading Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential ) {
                        sh 'docker tag chrostmarcin/nginx chrostmarcin/nginx:1.3'
                        sh 'docker push chrostmarcin/nginx:1.3'
                    }
                }    
            }
        }
    }
}

