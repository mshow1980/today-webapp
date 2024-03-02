pipeline{
    agent any
    tools{
        maven 'maven3'
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
    }

}