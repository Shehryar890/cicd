pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "malikshehryar/cicd"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Shehryar890/cicd.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([string(
                        credentialsId: 'sonartoken',
                        variable: 'SONAR_TOKEN'
                    )]) {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jenkins-token',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login \
                        -u $DOCKER_USER \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$IMAGE_TAG ."
                sh "docker tag $DOCKER_IMAGE:$IMAGE_TAG $DOCKER_IMAGE:latest"
            }
        }

        stage('Pull Image') {
            steps {
                sh "docker pull $DOCKER_IMAGE:$IMAGE_TAG"
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f cicd-app || true

                    docker run -d \
                    --name cicd-app \
                    -p 8080:8080 \
                    $DOCKER_IMAGE:$IMAGE_TAG
                '''
            }
        }

        stage('DAST Scan (OWASP ZAP)') {
            steps {
                sh '''
                    docker run --rm -t \
                    ghcr.io/zaproxy/zaproxy:stable \
                    zap-baseline.py \
                    -I \
                    -t http://host.docker.internal:8080
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh "docker push $DOCKER_IMAGE:$IMAGE_TAG"
                sh "docker push $DOCKER_IMAGE:latest"
            }
        }
    }

   post {
        always {
            sh 'docker rm -f cicd-app || true'
        }
    }
}
