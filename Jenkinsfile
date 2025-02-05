pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/python-app:${env.BUILD_NUMBER}"
        KUBE_CONFIG = "--kubeconfig=/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                script {
                    sh "trivy image --exit-code 1 --severity CRITICAL ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl ${KUBE_CONFIG} apply -f kubernetes/deployment.yaml
                        kubectl ${KUBE_CONFIG} apply -f kubernetes/service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
