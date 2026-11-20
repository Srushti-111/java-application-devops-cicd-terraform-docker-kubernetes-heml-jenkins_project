
pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    environment {
        DOCKER_IMAGE = 'srushhg/java-web-application'
        SONAR_HOST = 'http://3.6.90.197:9000'
        CONTAINER_NAME = 'javawebapp'
        APP_PORT = '8080'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Srushti-111/java-application-devops-cicd-terraform-docker-kubernetes-heml-jenkins_project.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
                        -Dsonar.projectKey=my-project \
                        -Dsonar.projectName=my-project \
                        -Dsonar.host.url=http://3.6.90.197:9000 \
                        -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:latest .
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set +x

                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push $DOCKER_IMAGE:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Docker Pull') {
            steps {
                sh '''
                    docker pull $DOCKER_IMAGE:latest
                '''
            }
        }

       stage('Docker Run / Deploy') {
    steps {
        sh '''
            docker rm -f javawebapp || true

            docker run -d \
            --name educationapp \
            --restart unless-stopped \
            -p 8081:8080 \
            srushhg/java-web-application

            docker ps
        '''
    }
}
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Docker application deployed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            echo 'Jenkins pipeline execution finished.'
        }
    }
}
