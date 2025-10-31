node {
    docker.image('node:18-alpine').inside {
        stage('Checkout') {
            checkout scm
        }

        stage('Install Dependencies') {
            sh 'npm ci'
        }

        stage('Snyk Scan') {
            snykSecurity(
                snykInstallation: 'Snyk-installation',
                snykTokenId: 'synk_id',
                severity: 'critical'
            )
        }

        stage('SonarQube Analysis') {
            def scannerHome = tool 'SonarQube-Scanner'
            withSonarQubeEnv('SonarQube-installations') {
                sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=nodejs-chat-app -Dsonar.sources=."
            }
        }

        stage('Docker Build & Deploy') {
            sh '''
                docker build -t vankorj/nodejs-chat-app:latest .
                docker-compose down
                docker-compose up -d
            '''
        }
    }
}
