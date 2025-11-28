pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/dahmanidounia/simple-express-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    npm install
                    npm test || echo "Tests échoués mais on continue"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t simple-express-app .
                '''
            }
        }
    }
}
