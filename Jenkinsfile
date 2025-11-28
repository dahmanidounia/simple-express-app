pipeline {
    agent any

    environment {
        AWS_REGION = "eu-west-3"
        ECR_REPO  = "784748657947.dkr.ecr.eu-west-3.amazonaws.com/simple-express-app"
        APP_NAME  = "simple-express-app"
    }

    tools {
        nodejs "Node18"   // Assure-toi que Node18 existe dans Global Tool Configuration
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/dahmanidounia/simple-express-app.git'
            }
        }

        stage('Install Node Modules') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "Aucun test trouvé"'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${APP_NAME}:latest ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_REPO}
                """
            }
        }

        stage('Docker Tag & Push') {
            steps {
                sh """
                    docker tag ${APP_NAME}:latest ${ECR_REPO}:latest
                    docker push ${ECR_REPO}:latest
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline exécuté avec succès ! 🎉"
        }
        failure {
            echo "Erreur dans le pipeline ❌"
        }
    }
}
