pipeline{
    agent any
    tools{
        jdk 'jdk21'
        nodejs 'node20'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git 'https://github.com/Gaetanneo/DevOps-Project-Swiggy-app.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' 
                    $SCANNER_HOME/bin/SonarQube \
                        -Dsonar.projectName=swiggy-app-gaetan \
                        -Dsonar.projectKey=swiggy-app-gaetan 
                    '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token-gaetan' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker'){   
                       sh "docker build -t swiggy ."
                       sh "docker tag swiggy gaetanneo/swiggy:latest "
                       sh "docker push gaetanneo/swiggy:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image gaetanneo/swiggy:latest > trivy.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh '''
                docker rm -f swiggy || true
                docker run -d --name swiggy -p 3000:3000 gaetanneo/swiggy:latest
                '''
            }
        }
    }
}
