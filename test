pipeline {
    agent any

    parameters {
        choice(name: 'ACTION', choices: ['Docker', 'Kubernetes', 'Both'], description: 'Select the action to perform')
        string(name: 'REGISTRY', defaultValue: '192.168.4.81:5000', description: 'Docker registry URL')
        string(name: 'IMAGE_NAME', defaultValue: 'helloworld', description: 'Name of the Docker image')
        string(name: 'IMAGE_TAG', defaultValue: "${env.BUILD_NUMBER}-${env.GIT_COMMIT.substring(0, 7)}", description: 'Tag for the Docker image')
        string(name: 'KUBE_CONFIG', defaultValue: '81conf', description: 'Kubernetes credentials ID')
        string(name: 'DEPLOYMENT_FILE', defaultValue: 'deploy.yaml', description: 'Kubernetes deployment file path')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Tamilarasand02/cicd-test-with-jenkins.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build the Maven project
                    sh 'mvn clean package'
                }
            }
        }

        // Conditional stage for Docker actions
        stage('Docker Build & Push') {
            when {
                expression { params.ACTION == 'Docker' || params.ACTION == 'Both' }
            }
            steps {
                script {
                    // Build Docker image with provided tag
                    docker.build("${params.REGISTRY}/${params.IMAGE_NAME}:${params.IMAGE_TAG}")

                    // Push Docker image to private registry
                    docker.withRegistry("http://${params.REGISTRY}") {
                        docker.image("${params.REGISTRY}/${params.IMAGE_NAME}:${params.IMAGE_TAG}").push()
                    }
                }
            }
        }

        // Conditional stage for Kubernetes actions
        stage('Update Kubernetes Manifests') {
            when {
                expression { params.ACTION == 'Kubernetes' || params.ACTION == 'Both' }
            }
            steps {
                script {
                    // Update the image tag in Kubernetes deployment YAML
                    sh """
                    sed -i 's|image: .*|image: ${params.REGISTRY}/${params.IMAGE_NAME}:${params.IMAGE_TAG}|g' $WORKSPACE/${params.DEPLOYMENT_FILE}
                    """
                }
            }
        }

        // Conditional stage for deploying to Kubernetes
        stage('Deploy to Kubernetes') {
            when {
                expression { params.ACTION == 'Kubernetes' || params.ACTION == 'Both' }
            }
            steps {
                script {
                    // Apply the updated YAML file to deploy the new image version
                    sh "kubectl --kubeconfig=${params.KUBE_CONFIG} apply -f $WORKSPACE/${params.DEPLOYMENT_FILE}"
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment successful with image tag: ${params.IMAGE_TAG}"
        }
        failure {
            echo "Build or deployment failed."
        }
    }
}
