pipeline {
    agent any
    
    stages{
        
       stage('SonarQube Analysis') {
         steps {
           script {
              // requires SonarQube Scanner 2.8+
              scannerHome = tool 'SonarScanner'
           }
           withSonarQubeEnv('SonarQube Server') {
             sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=python-app"
           }
      }
   }

        stage('Build Docker Image') {
            steps {
                script{
                    sh 'docker build -t sundarp1985/python-http-server .'
            }
        }
    }
        stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name python-app sundarp1985/python-http-server && sleep 10 && docker stop python-app'
                }
            }
        }
        stage('Push Image To Dockerhub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubpass')]) {
                    sh 'docker login -u sundarp1985 --password ${DockerHubpass}' }
                    sh 'docker push sundarp1985/python-http-server'
                }
            }
        }    
}
        post {
        always {
            // Always executed
                sh 'docker rm python-app'
        }
        success {
            // on sucessful execution
            sh 'docker logout'   
        }
    }
}
