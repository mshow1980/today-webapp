pipeline{
    agent any
    tools{
        maven 'maven3'
    }
    environment{
        APP_NAME = "today-webapp"
        RELEASE = "1.0.0"
        DOCKER_USER = "mshow1980"
        REGISTRY_CREDS = 'Docker-login'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = "JENKINS_API_TOKEN"
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
                    withSonarQubeEnv(credentialsId: 'SOnar-Token') {
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
                        withDockerRegistry(credentialsId: 'Docker-login')  {
                        docker_image = docker.build "${IMAGE_NAME}"   
                        }
                    }
                }
                stage('Pushing Image') {
                    steps{
                        script{
                        withDockerRegistry(credentialsId: 'Docker-login')  {
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
                            docker rmi ${IMAGE_NAME}:${IMAGE_TAG}
                            docker rmi ${IMAGE_NAME}:latest
                            """
                        }
                    }
                }
                stage('Updating K8 Manifest'){
                    steps{
                        script{
                            sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://3.238.106.128:8080/job/gitops-deployment/buildWithParameters?token=scion-scope'"
                        }
                    }
                }
            }
        }
    }