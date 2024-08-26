pipeline{
    agent{ label 'Jenkis-Agent'}
    tools{
        jdk 'Java17'
        mavem 'Maven3'
    }
    stages("Cleanuo Workspace"){
            steps{
            cleanWs()    
            }
    }

    stage("Checkout from SCM"){
        steps {
              git branch: 'main', credentialsId: 'github', url: 'https://github.com/Gsfrota/DevOps-Project1'
        }
    }
    stages("Build application"){
        step{
            sh "mvn clean package"
        }
    }
    stages("Test Application"){
        steps{
            sh "mvn test"
        }
    }    
}
