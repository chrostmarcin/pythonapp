pipeline {
    
    agent any
    
    triggers {
        pollSCM '*/30 * * * *'
    }

    environment {
        dockerImage =''
        registry = 'chrostmarcin/nginx'
        registryCredential ='docker_id'
        GIT_SSH_COMMAND = 'ssh -o StrictHostKeyChecking=no' // Skip host key checking
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
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                                  
                }
            }
        }

        stage ('Testing Container') {
            steps {
                script {
                    sh 'docker rm -f nginx 2>@1>/dev/null'
                    sh "docker run  -it --rm -d --name nginx -p 8181:80 chrostmarcin/nginx:'$BUILD_NUMBER'"
                    sh 'curl 192.168.68.120:8181'
                    sh 'docker rm -f nginx'
                }
            }    
        }
        
        
        stage ('Uploading Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential ) {
                        sh "docker tag chrostmarcin/nginx:'$BUILD_NUMBER' chrostmarcin/nginx:'$BUILD_NUMBER'"
                        sh "docker push chrostmarcin/nginx:'$BUILD_NUMBER'"
                    }
                }    
            }
        }


        stage ('Github clone the html yaml') {
            steps {
                script {

                    sh "rm -rf html/ 2>@1>/dev/null"    
                    sh "git clone https://github.com/chrostmarcin/html.git"
                    git branch: 'master', credentialsId: 'git_id', url: 'git@github.com:chrostmarcin/html.git'
                   
                    sh 'git config --global user.name "chrost.marcin"'
                    sh 'git config --global user.email "chrost.marcin@gmail.com"'
                    sshagent (credentials: ['git_id']) {
                        dir('html') {

                            sh "pwd"
                            sh "echo $BUILD_NUMBER"
                            sh "ls -alh"
                            sh "cat deployment.yaml"
                            sh "sed -i -e 's/nginx:.*/nginx:$BUILD_NUMBER/' deployment.yaml"
                            sh "cat deployment.yaml"
                            sh "cat index.html"
                            sh "sed -i -e 's/nginx.*/nginx:$BUILD_NUMBER plus 1/' index.html"
                            sh "cat index.html"
                            sh "git add . "
                            sh 'git commit -m "echo"'
                            sh 'git remote set-url origin git@github.com:chrostmarcin/html.git'
                            sh 'git push origin HEAD:master'
                        }
                    }
                }          
            }
        }  


          
    }
}

