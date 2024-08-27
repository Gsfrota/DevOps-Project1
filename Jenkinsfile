pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment{
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "gsfrota"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
      
    }

    stages {
            stage("Cleanup Workspace") { 
                steps {
                    cleanWs()
                }
            }

            stage("Checkout from SCM") { 
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Gsfrota/DevOps-Project1'
                }
            }

            stage("Build Application") { 
                steps {
                    sh "mvn clean package" 
                }
            }

            stage("Test Application") { 
                steps {
                    sh "mvn test"
                }
            }
            
            stage("SonarQube Analysis") { 
                steps {
                    script{
                        withSonarQubeEnv(credentialsId: 'jenkins-sonaqube-token'){
                            sh "mvn sonar:sonar"
                        }
                    } 
                }
            }
            stage("Quality Gate") { 
                steps {
                    script{
                        waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonaqube-token'
                    }
                }
            }
         //   stage("Verify Files") {
         //       steps { //usei essa verificação pois não estava encontrando o .war após o build
         //        sh "find . -name '*.war'ss"
         //       }
         //   }
            stage("Build & Push Docker Image") {
                steps {
                    script {
                        docker.withRegistry('',DOCKER_PASS) {
                            docker_image = docker.build "${IMAGE_NAME}"
                        }

                        docker.withRegistry('',DOCKER_PASS) {
                            docker_image.push("${IMAGE_TAG}")
                            docker_image.push('latest')
                        }
                    }
                }
            }
            stage("Trivy Scan"){
                script{
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image gsfrota/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
            }
            stage ("Cleanup Artifacts"){
                steps{
                    script{
                        sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker rmi ${IMAGE_NAME}:latest"
                    }
                }
            }
    
}
