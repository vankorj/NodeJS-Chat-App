node {
    env.DOCKERHUB_CREDENTIALS = 'docker-id'
    env.IMAGE_NAME = 'vankorj/nodejs-chat-app'

    stage('Checkout') {
        checkout scm
    }

    stage('Snyk SCA & SAST') {
        withCredentials([string(credentialsId: 'synk_id', variable: 'SNYK_TOKEN')]) {
            sh 'npm ci'
            sh 'npm i -g snyk@latest'
            sh 'export SNYK_TOKEN=${SNYK_TOKEN} && snyk auth ${SNYK_TOKEN}'

            // Run Snyk test (fails if high/critical vulnerabilities exist)
            sh 'snyk test --all-projects --severity-threshold=high'

            // Monitor project in Snyk UI
            sh 'snyk monitor --all-projects'
        }
    }

    stage('SonarQube Analysis') {
        def scannerHome = tool 'SonarQube-Scanner'
        withSonarQubeEnv('SonarQube-installations') {
            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=nodejs-chat-app -Dsonar.sources=."
        }
    }

    stage('Docker Build') {
        echo "Building Docker image ${env.IMAGE_NAME}..."
        app = docker.build("${env.IMAGE_NAME}")
        app.tag("latest")
    }

    stage('Push to DockerHub') {
        docker.withRegistry('https://registry.hub.docker.com', env.DOCKERHUB_CREDENTIALS) {
            app.push('latest')
        }
    }

    stage('Deploy') {
        echo 'Deploying NodeJS Chat App using docker-compose...'
        sh '''
            docker-compose down
            docker-compose up -d
            docker ps
        '''
        echo 'Deployment completed successfully!'
    }

    stage('Post Build') {
        echo 'Pipeline completed (success or fail).'
    }
}
