pipeline{
    agent any
    tools{
        maven 'maven3'
    }
    environment{
        APP_NAME = "today-webapp"
        RELEASE = "1.0.0"
        DOCKER_USER = "mshow1980"
        REGISTRY_CREDS = 'docker-login'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage('CleanWorkSpace') {
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            steps{
                script{
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mshow1980/today-webapp.git']])
                }
            }
        }
        stage ('OWASP Dependency-Check Vulnerabilities') {
            steps {
                script{
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'OWASP-DC'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                }
            }
        }
        stage('TRIVY FS SCAN'){
            steps{
                script{
                    sh 'trivy fs .'
                }
            }
        }
        stage('Test Application'){
            steps{
                script{
                    sh 'mvn test'
                }
            }
        }
                stage('mvn build'){
            steps{
                script{
                    sh 'mvn package'
                }
            }
        }
        stage('SOnarqube Analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'SOnar-token') {
                        sh 'mvn sonar:sonar'
                        }
                    }
                }
            }
        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'SOnar-Token'
                    }
                }
            }
        stage('Build Docker Image') {
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-login')  {
                    docker_image = docker.build "${IMAGE_NAME}"   
                        }
                    withDockerRegistry(credentialsId: 'docker-login')  {
                    docker_image.push("${BUILD_NUMBER}")
                    docker_image.push('latest')
                            }
                        }
                    }
                }
        stage('Delete Images & logout Docker'){
            steps{
                script{
                    sh """
                        docker rmi ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker rmi ${IMAGE_NAME}:latest
                        """
                        }
                    }
                }
            }
        }
