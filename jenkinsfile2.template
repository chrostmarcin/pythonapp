pipeline{
    agent any
    tools { maven "maven1"}
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages{
        stage ('Maven clean'){
            steps {
                sh 'mvn clean'
            }


        }

        stage ('Maven package'){
            steps {
                sh 'mvn package'
            }


        }

        stage ('Sonarqube analysis and testing'){
            steps{
                script{
                withSonarQubeEnv('sonarqube'){
                    sh 'mvn clean package sonar:sonar'
                }
                }
            }
    }    
   stage ("Quality Gate Check") {
            steps {
                script {
                timeout(time: 1, unit: 'HOURS') { 
                def qg = waitForQualityGate() 
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
                }
                }
            }
    }
    stage ('Docker build and push'){
   
           steps{
             withDockerRegistry([ credentialsId: "Docker_creds", url: "https://index.docker.io/v1/" ]){
               sh 'docker build -t devopstrainingschool/knote-eagle1:$BUILD_NUMBER . -f Dockerfile'
               sh 'docker push devopstrainingschool/knote-eagle1:$BUILD_NUMBER'
               sh 'rm -rf eagle1-yaml-file'
             }
           }
    }

    stage('Github clone the eagle1-yaml-file'){
            steps {
                script{
                  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
              sh "git clone https://github.com/thedevopsguru1/eagle1-yaml-file.git"
              
            
             
                    }
                  }
                }
            }
        }

        stage('Make yaml files changes & push them to Guthub'){
            steps {
                script{
                  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    
                     sh """
                    git config --global user.name "thedevopsguru1"
                    git config --global user.email "yannickparker1984@gmail.com" """
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                      dir('eagle1-yaml-file') {
                        sh "pwd"
                        sh "echo $BUILD_NUMBER"
                        sh "ls -alh"
                        sh "cat webapp.yaml"
                        sh "sed -i -e 's+knote-eagle1.*+knote-eagle1:${env.BUILD_NUMBER}+g'  webapp.yaml"
                        sh "cat webapp.yaml"
                        sh " git add . "
                        sh " git commit -m 'Updated the deployment file'"
                        sh "git push http://$GIT_USERNAME:$GIT_PASSWORD@github.com/thedevopsguru1/eagle1-yaml-file.git main"
             
                      }
             
                    }
                  }
                }
            }
        }








    }




}