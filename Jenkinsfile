pipeline {
    environment {
        IMAGE_NAME = "staticwebsite"
        APP_CONTAINER_PORT = "5000"
        APP_EXPOSED_PORT = "8082"
        IMAGE_TAG = "latest"
        STAGING = "chocoapp-jenkins-staging"
        PRODUCTION = "chocoapp-jenkins-prod"
        DOCKERHUB_ID = "docker19191919"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
    }

    agent none

    stages {
        stage('Build image') {
            agent { label 'Windows docker worker node' }
            steps {
                script {
                    def dockerHubIdLower = DOCKERHUB_ID.toLowerCase() // Conversion en minuscules
                    echo "DOCKERHUB_ID: ${dockerHubIdLower}"
                    bat """
                        docker build -t ${dockerHubIdLower}/%IMAGE_NAME%:%IMAGE_TAG% .
                    """
                }
            }
        }
        stage('Run container based on built image') {
            agent { label 'Windows docker worker node' }
            steps {
                script {
                    bat """
                        echo Cleaning existing container if exist
                        docker ps -a | findstr %IMAGE_NAME% && docker rm -f %IMAGE_NAME%
                        docker run --name %IMAGE_NAME% -d -p %APP_EXPOSED_PORT%:%APP_CONTAINER_PORT% -e PORT=%APP_CONTAINER_PORT% ${DOCKERHUB_ID}/%IMAGE_NAME%:%IMAGE_TAG%
                        ping 127.0.0.1 -n 6 > nul

                    """
                }
            }
        }
        stage('Test image') {
            agent { label 'Windows docker worker node' }
            steps {
                script {
                    bat """
                        docker ps -a
                        curl 172.28.128.123:8082 | findstr Dimension
                    """
                }
            }
        }
        stage('Clean container') {
            agent { label 'Windows docker worker node' }
            steps {
                script {
                    bat """
                        docker stop %IMAGE_NAME%
                        docker rm %IMAGE_NAME%
                    """
                }
            }
        }
        stage('Login and Push Image on Docker Hub') {
            agent { label 'Windows docker worker node' }
            steps {
                script {
                    def dockerHubIdLower = DOCKERHUB_ID.toLowerCase() // Conversion en minuscules
                    bat """
                        echo ${DOCKERHUB_PASSWORD} | docker login -u ${dockerHubIdLower} --password-stdin
                        docker push ${dockerHubIdLower}/%IMAGE_NAME%:%IMAGE_TAG%
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        }
    }
}
