node {
    env.DOCKERHUB_CREDENTIALS = 'docker-id'
    env.IMAGE_NAME = 'vankorj/nodejs-chat-app'

    stage('Checkout') {
        checkout scm
    }

    stage('Install Dependencies') {
        sh 'npm ci'
    }

    stage('Verify Node Modules') {
        sh '''
            ls -l
            ls -l node_modules || echo "node_modules missing"
            npm ci
        '''
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
