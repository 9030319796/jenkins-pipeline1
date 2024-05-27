pipeline{
    agent any
    tools{
        jdk 'jdk'
        python3 'python3'
    }
    environment {
        SCANNER_HOME=tool 'SONAR_HOME'
    }
    stages {
        stage('Workspace Cleaning'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/9030319796/jenkins-pipeline1.git'
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar_scanner -Dsonar.projectName=Python \
                    -Dsonar.projectKey=Python \
                    '''
                }
            }
        }
        stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: '731696616771c546da7b35333248cfaa0835e0f1' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "apt-get install python3"
            }
        }
        // stage('OWASP DP SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    sh 'docker build -t 9030319796/python-http-server .'
                }
            }
        }
        stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name python-app 9030319796/python-http-server && sleep 10 && docker stop python-app'
                }
            }
        }
        stage('Push Image To Dockerhub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubpass')]) {
                    sh 'docker login -u 9030319796 --password ${DockerHubpass}' }
                    sh 'docker push 9030319796/python-http-server:latest'
                }
            }
        }    
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image 9030319796/python-http-server:latest > trivyimage.txt" 
            }
        }
        // stage('Deploy to Kubernetes'){
        //     steps{
        //         script{
        //             dir('Kubernetes') {
        //                 withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                         sh 'kubectl apply -f deployment.yml'
        //                         sh 'kubectl apply -f service.yml'
        //                         sh 'kubectl get svc'
        //                         sh 'kubectl get all'
        //                 }   
        //             }
        //         }
        //     }
        // }
    }
    // post {
    //  always {
    //     emailext attachLog: true,
    //         subject: "'${currentBuild.result}'",
    //         body: "Project: ${env.JOB_NAME}<br/>" +
    //             "Build Number: ${env.BUILD_NUMBER}<br/>" +
    //             "URL: ${env.BUILD_URL}<br/>",
    //         to: 'penta.sundar85@gmail.com',
    //         attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    //     }
    // }
}
