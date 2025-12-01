pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    tools {
        maven 'maven3'
    }

    environment {
        SONAR_URL = "http://13.203.195.158:9000"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning GitHub Repo...'
                git branch: 'main', url: 'https://github.com/Jayesh7744/mindcircuit13.git'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh """
                            sonar-scanner \
                              -Dsonar.projectKey=test \
                              -Dsonar.sources=./src \
                              -Dsonar.host.url=${SONAR_URL} \
                              -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build Artifact') {
            steps {
                echo 'Building Maven Artifact...'
                sh 'mvn clean package'
            }
        }

        stage('Docker Image Build') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t jayesh7744/ultimate-cicd:${BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub', variable: 'DOCKERHUB_TOKEN')]) {
                        sh """
                            echo "${DOCKERHUB_TOKEN}" | docker login -u "jayesh7744" --password-stdin
                            docker push jayesh7744/ultimate-cicd:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
