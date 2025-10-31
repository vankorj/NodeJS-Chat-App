pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'docker-id'
        IMAGE_NAME = 'vankorj/nodejs-chat-app'
    }

    stages {
        stage('Cloning Git') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        echo 'Running SonarQube analysis...'
                        def scannerHome = tool 'SonarQube-Scanner'
                        withSonarQubeEnv('SonarQube-installations') {
                            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=nodejs-chat-app -Dsonar.sources=."
                        }
                    }
                }
            }
        }

        stage('SAST-TEST') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        snykSecurity(
                            snykInstallation: 'Snyk-installation',
                            snykTokenId: 'synk_id',
                            severity: 'critical'
                        )
                    }
                }
            }
        }

        stage('BUILD-AND-TAG') {
            agent { label 'CWEB-2140-60-Appserver-Korbin' }
            steps {
                script {
                    echo "Building Docker image ${IMAGE_NAME}..."
                    // Build Docker image; npm ci runs inside the Dockerfile
                    def app = docker.build("${IMAGE_NAME}:latest", ".")
                }
            }
        }

        stage('RUN-CONTAINER') {
            agent { label 'CWEB-2140-60-Appserver-Korbin' }
            steps {
                script {
                    echo "Running Docker container..."
                    // Remove any previous container to avoid name conflict
                    sh '''
                        docker rm -f nodejs-chat-app-container || true
                        docker run -d --name nodejs-chat-app-container -p 3700:3700 vankorj/nodejs-chat-app:latest
                    '''
                }
            }
        }

        stage('POST-TO-DOCKERHUB') {
            agent { label 'CWEB-2140-60-Appserver-Korbin' }
            steps {
                script {
                    echo "Pushing image ${IMAGE_NAME}:latest to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDENTIALS) {
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('DEPLOYMENT') {
            agent { label 'CWEB-2140-60-Appserver-Korbin' }
            steps {
                echo 'Starting deployment using docker-compose...'
                script {
                    sh '''
                        docker-compose down
                        docker-compose up -d
                        docker ps
                    '''
                }
                echo 'Deployment completed successfully!'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed (success or fail).'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
