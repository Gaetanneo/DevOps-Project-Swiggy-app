pipeline{
    agent any
    tools{
        jdk 'JDK21'
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
                git branch: 'master', url: 'https://github.com/Gaetanneo/DevOps-Project-Swiggy-app.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-scanner') {
                    sh ''' 
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
        stage('Trivy Filesystem Scan') {
            steps {
                sh "trivy fs . --exit-code 0 --severity HIGH,CRITICAL -f table -o trivy-fs-report.txt"
                archiveArtifacts artifacts: 'trivy-fs-report.txt', allowEmptyArchive: true
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
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image gaetanneo/swiggy:latest --exit-code 0 --severity HIGH,CRITICAL -f table -o trivy-image-report.txt"
                archiveArtifacts artifacts: 'trivy-image-report.txt', allowEmptyArchive: true
            }
        }
        // stage('Deploy to container'){
        //     steps{
        //         sh '''
        //         docker rm -f swiggy || true
        //         docker run -d --name swiggy -p 3000:3000 gaetanneo/swiggy:latest
        //         '''
        //     }
        // }
    }
}
