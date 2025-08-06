pipeline {
    agent any

    environment {
        IMAGE_NAME = "ahmadbilal124/python-jenkins"
        CONTAINER_NAME = "flaskapp"
        APP_PORT = "5000"
        DEPLOY_USER = "ubuntu"
        DEPLOY_HOST = "18.191.135.178"
    }

    options {
        skipStagesAfterUnstable()  // avoid running later stages on failure
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/Ahmad-Bilal-124/python-jenkins.git'
            }
        }

        stage('Install Python Dependencies') {
            steps {
                sh '''
                    pip3 install -r python-app/requirements.txt
                    pip3 install pytest pylint
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest python-app/test_app.py'
            }
        }

        stage('Lint Code') {
            steps {
                sh 'pylint python-app/app.py || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub_credentials', variable: 'DOCKER_PASS')]) {
                    script {
                        def DOCKER_USER = "ahmadbilal124"
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $IMAGE_NAME:$BUILD_NUMBER
                        """
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                withCredentials([string(credentialsId: 'github_credentials', variable: 'SSH_PRIVATE_KEY')]) {
                    sh """
                        echo "$SSH_PRIVATE_KEY" > key.pem
                        chmod 600 key.pem

                        ssh -i key.pem -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST '
                            docker pull $IMAGE_NAME:$BUILD_NUMBER &&
                            docker stop $CONTAINER_NAME || true &&
                            docker rm $CONTAINER_NAME || true &&
                            docker run -d -p $APP_PORT:5000 --name $CONTAINER_NAME $IMAGE_NAME:$BUILD_NUMBER
                        '

                        rm key.pem
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployed: http://$DEPLOY_HOST:$APP_PORT"
        }
        failure {
            echo "❌ Build failed. Fix and commit again to resume from failed stage."
        }
    }
}
