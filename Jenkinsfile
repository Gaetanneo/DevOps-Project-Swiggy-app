pipeline{
    agent any
    tools{
        jdk 'JDK21'
        nodejs 'node20'
    }
    environment {
        SONAR_PROJECT_KEY = 'swiggy-app-gaetan'
        SONAR_URL      = 'https://sonarqube.devopspro.cloud'
        EC2_HOST       = '107.21.136.9'
        EC2_USER       = 'ubuntu'
        IMAGE          = 'gaetanneo/swiggy:latest'
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

        stage('Debug Sonar Environment') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    sh """
                        echo "=== JAVA ==="
                        which java || true
                        java -version || true
                        echo "JAVA_HOME=\$JAVA_HOME"

                        echo ""
                        echo "=== SONAR ==="
                        echo "${scannerHome}"
                        ls -l ${scannerHome}/bin

                        echo ""
                        echo "=== SONAR VERSION ==="
                        ${scannerHome}/bin/sonar-scanner --version || true
                    """
               }
           }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=swiggy-app-gaetan \
                            -Dsonar.projectName=swiggy-app-gaetan \
                            -Dsonar.sources=.
                        """
                    }
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
        // stage('OWASP FS SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                trivy fs . \
                  --severity HIGH,CRITICAL \
                  --format table
                '''
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-creds') {   
                       sh "docker build -t swiggy ."
                       sh "docker tag swiggy gaetanneo/swiggy:latest "
                       sh "docker push gaetanneo/swiggy:latest "
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image gaetanneo/swiggy:latest \
                  --severity HIGH,CRITICAL \
                  --format table \
                '''
             }
        }
        stage('Deploy to EC2 Host'){
            steps{
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictkHostKeyChecking=no ${EC2_USER}@{EC2_HOST} "
                            docker pull ${IMAGE} &&
                            docker rm -f swiggy || true &&
                            docker run -d --name swiggy -p 3000:3000 gaetanneo/swiggy:latest
                        "
                    '''
            }
        }
    }
}
