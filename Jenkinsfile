pipeline {
    agent any
    tools {

        nodejs 'node21'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('Docker_hub')
    }
    
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Abbyabiola/Amazonproject.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=amazonproject -Dsonar.projectKey=amazonproject"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
                }
            }
        }
        
        stage('NPM') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Trivy File Scan') {
            steps {
                script {
                    sh '/usr/local/bin/trivy fs . > trivy_result.txt'
                }
            }
        }
    
        stage('Login to DockerHUB') {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    echo 'Login Succeeded'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t abimbola1981/amazonproject:latest .' 
                    echo "Image Build Successfully"
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    sh '/usr/local/bin/trivy image abimbola1981/amazonproject:latest > trivy_image_result.txt'
                    sh 'pwd'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    sh 'docker push abimbola1981/amazonproject:latest'
                    echo "Push Image to Registry"
                }
            }
        }
        stage('Containerization Deployment') {
            steps {
                script {
                    def containername = 'amazonproject'
                    def isRunning = sh(script: "docker ps -a | grep ${containername}", returnStatus: true)
                    if (isRunning == 0) {
                        sh "docker stop ${containername}"
                        sh "docker rm ${containername}"
                    }
                    sh "docker run -d -p 3000:3000 --name ${containername} abimbola1981/amazonproject:latest"
                }
                    
              
            }
        }
    }
}
