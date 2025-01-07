pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE_NAME = 'votre-repo/ebanking-app'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        ORACLE_CLOUD_CREDENTIALS = credentials('oracle-cloud-credentials')
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                 git branch: 'main',
                    credentialsId: 'git-credentials',
                    url: 'https://github.com/votre-repo/ebanking.git'
            }
        }
        
        stage('Build Maven') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }
        
        stage('Tests') {
            steps {
                bat 'mvn test'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    bat """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                        docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    bat """
                        docker login -u ${DOCKER_HUB_CREDENTIALS_USR} -p ${DOCKER_HUB_CREDENTIALS_PSW}
                        docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        docker push ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }
        
        stage('Deploy to Oracle Cloud') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'oracle-cloud-key', variable: 'ORACLE_KEY_FILE')]) {
                        bat """
                            oci setup config --key-file %ORACLE_KEY_FILE% --region eu-frankfurt-1
                            oci container instance create --compartment-id ${ORACLE_COMPARTMENT_ID} \
                                --image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} \
                                --shape VM.Standard.E2.1 \
                                --subnet-id ${ORACLE_SUBNET_ID}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
            bat "docker rmi ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
        }
        failure {
            echo 'Pipeline execution failed!'
        }
        always {
            bat 'docker logout'
        }
    }
}