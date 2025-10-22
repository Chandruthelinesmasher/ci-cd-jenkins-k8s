pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-cicd"
        ACR_NAME = "chanacr" // replace with your actual ACR name
        ACR_LOGIN = "chanacr.azurecr.io" // replace with your actual ACR login server
    }

    stages {

        stage('Checkout') {
            steps {
                echo "✅ Code checked out automatically from SCM"
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER ./app'
                }
            }
        }

        stage('Scan Image with Trivy') {
            steps {
                sh 'trivy image $IMAGE_NAME:$BUILD_NUMBER || true'
            }
        }

        stage('Push to Azure Container Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login $ACR_LOGIN -u $USER --password-stdin
                        docker tag $IMAGE_NAME:$BUILD_NUMBER $ACR_LOGIN/$IMAGE_NAME:$BUILD_NUMBER
                        docker push $ACR_LOGIN/$IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/flask-app flask-container=$ACR_LOGIN/$IMAGE_NAME:$BUILD_NUMBER -n dev
                kubectl rollout status deployment/flask-app -n dev
                '''
            }
        }

        stage('Notify') {
            steps {
                echo "✅ Build #${BUILD_NUMBER} deployed successfully!"
            }
        }
    }
}
