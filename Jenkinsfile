pipeline {
    agent any 

    environment  {
        AWS_ACCOUNT_ID = credentials('ACCOUNT_ID')
        AWS_ECR_REPO_NAME = "checkoutservice"
        AWS_DEFAULT_REGION = 'us-east-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }

    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Nani5763/Ten-microservices-ecommerce-k8s.git'
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    dir('src/checkoutservice') {
                        sh 'docker system prune -f'
                        sh 'docker container prune -f'
                        sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
                    }
                }
            }
        }
        stage("ECR Image Pushing") {
           steps {
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                 script {
                     sh '''
                         aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | \
                         docker login --username AWS --password-stdin ${REPOSITORY_URI}
                         docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}
                         docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}
                      '''
	         }
              }
           }
        }
}
