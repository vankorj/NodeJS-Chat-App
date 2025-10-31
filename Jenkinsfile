node {
    def nodeHome = tool name: 'Node18', type: 'NodeJS'
        env.PATH = "${nodeHome}/bin:${env.PATH}"

    stage('Install Dependencies') {
        sh 'npm ci'
    }

    stage('Snyk SCA & SAST') {
        script {
            snykSecurity(
                snykInstallation: 'Snyk-installation',
                snykTokenId: 'synk_id',
                severity: 'critical'
            )
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
