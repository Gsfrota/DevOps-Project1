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




            stage("Verify WAR Files") {
                steps {
                    script {
                        // Define o diretório onde o arquivo .war deve estar
                        def targetDir = 'target'

                        // Verifica se o diretório existe
                        if (fileExists(targetDir)) {
                            // Lista o conteúdo do diretório para verificar se o .war está presente
                            sh "ls -al ${targetDir}"
                            
                            // Verifica se há arquivos .war no diretório
                            def warFiles = sh(script: "ls ${targetDir}/*.war", returnStatus: true)
                            if (warFiles != 0) {
                                error "Nenhum arquivo .war encontrado no diretório ${targetDir}."
                            }
                        } else {
                            error "O diretório ${targetDir} não existe."
                        }
                    }
                }
            }




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
            }
    
}
