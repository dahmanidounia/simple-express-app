properties([
    pipelineTriggers([
        githubPush()   // Active le webhook GitHub
    ])
])

pipeline {
    agent any

    tools {
        nodejs "node17"
    }

    environment {
        AWS_REGION      = "us-east-1"
        AWS_ACCOUNT_ID  = "784748657947"
        ECR_REPOSITORY  = "rouge-java"
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
        ECR_IMAGE_URI   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"

        EC2_HOST        = "54.235.20.78"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/dahmanidounia/simple-express-app.git'
            }
        }

        stage('Load AWS Credentials') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws'
                ]]) {
                    sh 'echo AWS credentials loaded'
                }
            }
        }

        stage('Build & Test') {
            steps {
                sh """
                    npm install
                    echo Tests skipped (no test script)
                """
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${ECR_IMAGE_URI} .
                """
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws'
                ]]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} \
                            | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                        docker push ${ECR_IMAGE_URI}
                    """
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                sshagent(['ec2-user']) {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws'
                    ]]) {

                        sh """
                            ssh -o StrictHostKeyChecking=no ec2-user@${EC2_HOST} '

                                export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                                export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                                export AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}

                                # Stop and remove existing container
                                docker stop app || true
                                docker rm app || true

                                # Clean system
                                docker system prune -af || true

                                # Login to ECR
                                aws ecr get-login-password --region ${AWS_REGION} \
                                    | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                                # Pull latest image
                                docker pull ${ECR_IMAGE_URI}

                                # Run new container
                                docker run -d --name app -p 3000:3000 ${ECR_IMAGE_URI}
                            '
                        """
                    }
                }
            }
        }
    }
}
