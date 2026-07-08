pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = '121846058083'
        AWS_REGION     = 'us-east-1'
        ECR_REPO       = 'bookmyshow-app'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        ECR_URI        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rohanj722/Book-My-Show.git'
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} -f bookmyshow-app/Dockerfile bookmyshow-app"
            }
        }
        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-ecr-credentials', region: "${AWS_REGION}") {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                        docker push ${ECR_URI}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Deploy to EKS via Ansible') {
            steps {
                sh "ansible-playbook ansible-k8s-deploy.yaml"
            }
        }
    }
}
