pipeline {
    agent any
    
    environment{
        SONAR_HOME= tool 'sonar'
    }
    stages {
        stage('clean-ws') {
            steps {
                echo 'Clean workspace'
                cleanWs()
            }
        }
        stage('Git-checkout') {
            steps {
                echo 'Clone code from git'
                git changelog: false, poll: false, url: 'https://github.com/shubham9511s/vite_multilanguage_app.git'
            }
        }
        stage('static code quality check') {
            steps {
                echo 'Sonar scan start'
                withSonarQubeEnv('sonar-server') {
                    sh"$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=multivite-app -Dsonar.projectKey=multivite-app"
                    
                }
            }
        }
        stage('Trivy file scan') {
            steps {
                echo 'file scan start'
                trivy repo --format table -o fs-report.txt https://github.com/shubham9511s/vite_multilanguage_app.git
            }
        }
      stage('Build and Push ,Scan Docker Image') {
       environment {
        DOCKER_IMAGE = "shubhamshinde2025/ultimate-cicd:${BUILD_NUMBER}"
      }
      steps {
        script {
               withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                   
                     sh 'docker build -t ${DOCKER_IMAGE} .'
                      def dockerImage = docker.image("${DOCKER_IMAGE}")
                       trivy image --format table -o trivy-image-report.txt dockerImage
                       dockerImage.push()
                
               }
        }
      }
    }
       stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "vite_multilanguage_app"
            GIT_USER_NAME = "shubham9511s"
        }
        steps {
           withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
               
               sh '''
                    git config user.email "shubham.xyz@gmail.com"
                    git config user.name "Shubham Shinde"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" public/deployment.yml
                    git add public/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                '''
           }
           
        }
    }
    
       
        
        
        
    }
}
