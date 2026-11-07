pipeline {
    agent any

    environment {
        IMAGE_NAME = "srikantheggoju/secure-k8s-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=secure-k8s-app \
                    -Dsonar.projectName=secure-k8s-app \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t srikantheggoju/secure-k8s-app:${BUILD_NUMBER} .
                docker tag srikantheggoju/secure-k8s-app:${BUILD_NUMBER} srikantheggoju/secure-k8s-app:latest
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image srikantheggoju/secure-k8s-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
