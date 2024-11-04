pipeline {
    agent any
   
    environment {
        REGISTRY = "192.168.4.81:5000"
        IMAGE_NAME = "helloworld"
        IMAGE_TAG = "v1"
        KUBECONFIG = credentials('kubeconfig') // Kubernetes credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tamilarasan002/webhook-test.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Build Docker image with provided tag
                    docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")

                    // Push Docker image to private registry
                    docker.withRegistry("http://${REGISTRY}") {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply the updated YAML file to deploy the new image version
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f $WORKSPACE/deploy.yaml"
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment successful with image tag: ${IMAGE_TAG}"
        }
        failure {
            echo "Build or deployment failed."
        }
    }
}

