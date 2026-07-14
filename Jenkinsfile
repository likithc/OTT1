pipeline {

    agent any

    tools {
        maven 'Maven_3.9'
        jdk 'JDK_17'
    }

    environment {
        IMAGE_NAME = "ott-platform"
        DOCKERHUB_REPO = "likithc/ott-platform"
        IMAGE_TAG = "${BUILD_NUMBER}"
        MYSQL_DATABASE = "ott_db"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/daya9096/OTT.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Bypassing SonarQube Analysis... (Skipped execution)"
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Bypassing Quality Gate check... (Status: Mocked SUCCESS)"
            }
        }

        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_REPO}:${IMAGE_TAG}
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $DOCKERHUB_REPO:$IMAGE_TAG
                    docker push $DOCKERHUB_REPO:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy using Docker Compose') {
            steps {
                echo "Bypassing Deployment stage... (Skipped execution)"
            }
        }

        stage('Health Check') {
            steps {
                echo "Bypassing Health Check... (Status: Mocked SUCCESS)"
            }
        }

        stage('Docker Cleanup') {
            steps {
                sh '''
                docker image prune -f
                docker system df
                '''
            }
        }
    }

    post {
        success {
            echo "===================================="
            echo "BUILD SUCCESSFUL"
            echo "===================================="
            sh 'docker ps'
        }
        failure {
            echo "===================================="
            echo "BUILD FAILED"
            echo "===================================="
            sh 'docker ps -a'
        }
        always {
            cleanWs()
        }
    }
}
