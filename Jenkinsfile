pipeline {
    agent any

    environment {
        IMAGE_NAME = "ahmadcloudworks/python-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ahmad-bilal-124/flask-ci-pipeline.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r app/requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest app/test_app.py'
            }
        }

        stage('Code Analysis') {
            steps {
                sh 'pip3 install pylint'
                sh 'pylint app/app.py || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['your-ssh-key']) {
                    sh '''
                    ssh user@remote-server "
                        docker pull $IMAGE_NAME:$BUILD_NUMBER &&
                        docker stop flaskapp || true &&
                        docker rm flaskapp || true &&
                        docker run -d -p 5000:5000 --name flaskapp $IMAGE_NAME:$BUILD_NUMBER
                    "
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Build failed. Fix and push again to retry.'
        }
    }
}
