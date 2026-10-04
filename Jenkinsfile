pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"          // change to your region
        AWS_ACCOUNT_ID = "123456789012"        // change to your account ID
        ECR_REPOSITORY = "php-app-demo"
        IMAGE_URI      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:latest"
    }


        stage('Build Docker Image') {
            steps {
                sh 'docker build -t php-app-demo .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f php-test || true
                docker run -d --name php-test -p 8081:80 php-app-demo
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                sleep 5
                curl -f http://localhost:8081
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag php-app-demo:latest $IMAGE_URI'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_URI'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f php-test || true'
        }
        success { echo 'Image pushed to ECR successfully' }
        failure { echo 'Pipeline failed' }
    }
}
