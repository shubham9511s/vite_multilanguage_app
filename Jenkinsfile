pipeline { 
    agent any
    
    environment{
        SONAR_HOME= tool 'sonar'
    }
       tools {
        nodejs 'node20' 
    }
     stages {
        stage('clean-ws') {
            steps {
                echo 'Clean workspace'
                cleanWs()
            }
        }
        /*stage('Git-checkout') {
            steps {
                echo 'Clone code from git'
                git url: 'https://github.com/shubham9511s/vite_multilanguage_app.git'
            }
        }*/
        stage('static code quality check') {
            steps {
                echo 'Sonar scan start static code quality test'
                withSonarQubeEnv('sonar-server') {
                    sh"$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=multivite-app -Dsonar.projectKey=multivite-app"
                    
                }
            }
        }
        stage('OWASP Dependency-Check Scan') {
            steps {
                    echo'OWASP Dependency-Check Scan start'
                    dependencyCheck additionalArguments: '--scan ./, odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        } 
        stage('Trivy File scan') {
            steps {
                echo 'file scan start'
                 sh 'trivy fs . '
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
                     sh'trivy image --format table ${DOCKER_IMAGE}'
                      def dockerImage = docker.image("${DOCKER_IMAGE}")
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
           withCredentials([string(credentialsId:'github-token', variable:'TOKEN')]) {
               
               sh '''
                    git config user.email "shubham.xyz@gmail.com"
                    git config user.name "Shubham Shinde"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" public/deployment.yml
                    git add .
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                '''
           }
           
        }
    }
    
       
        
        
        
    }
    post{
        always {
            emailext(
                subject:"pipeline Status: ${BUILD_NUMBER}",
                body:''' <html>
                            <body>
                                  <p>Build Status: ${BUILD_STATUS}</p>
                                  <p>Build Number: ${BUILD_NUMBER}</p>
                                  <p>Check the <a href="${BUILD_URL}">Console output</a>.</p>
                            </body>
                            </html>''' ,
                            to: 'shubham.ssc100@gmail.com',
                            from:'jenkins@example.com',
                            replyTo:'jenkins@example.com',
                            mimeType:'text/html',
                          
            )
        }
    }


}
