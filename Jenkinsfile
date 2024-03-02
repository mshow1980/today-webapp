pipeline{
    agent any
    tools{
        maven 'maven3'
    }
    environment{
        APP_NAME = "today-webapp"
        RELEASE = "1.0.0"
        DOCKER_USER = "docker-login"
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
        stage('maven clean install'){
            steps{
                script{
                    sh "mvn clean install package"
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
        stage('Test Application'){
            steps{
                script{
                    sh 'mvn test'
                }
            }
        }
        stage('SOnarqube Analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'Jenkins-Token') {
                        sh 'mvn sonar:sonar'
                        }
                    }
                }
            }
            stage('Quality Gate'){
                steps{
                    script{
                        waitForQualityGate abortPipeline: false, credentialsId: 'Jenkins-Token'
                    }
                }
            }
            stage('Docker Build & login'){
                steps{
                    script{
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
            stage('Docker Push'){
                steps{
                    script{
                        withCredentials([usernamePassword(credentialsId: 'Docker-login', passwordVariable: 'docker-pass', usernameVariable: 'docker-login')]) {
                    sh """
                        docker login -u "${DOCKER_USER} -p $docker-pass
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                        """
                        }
                    }
                }
            }
        }
    }