pipeline {
    agent any

    environment {
        IMAGE_NAME = "srikantheggoju/secure-k8s-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonarqube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=secure-k8s-app \
                        -Dsonar.projectName=secure-k8s-app \
                        -Dsonar.sources=.
                        """
                    }
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
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                trivy image --skip-version-check \
                ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply --validate=false -f k8s/deployment.yaml
                kubectl apply --validate=false -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful'
        }

        failure {
            echo '❌ Pipeline Failed'
        }
    }
}
