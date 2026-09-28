pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo 'Checking out source code...'

                git(
                    branch: 'main',
                    url: 'https://github.com/Vamshi420/mindcircuit17d.git'
                )
            }
        }

        stage('Build Artifact') {
            steps {
                echo 'Building Maven artifact...'

                sh 'mvn clean package'
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo 'Starting SonarQube analysis...'

                withSonarQubeEnv(
                    installationName: 'SonarQube',
                    credentialsId: 'sonarqube'
                ) {
                    sh '''
                        mvn verify \
                        org.sonarsource.scanner.maven:sonar-maven-plugin:5.1.0.4751:sonar
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate...'

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('BUILD DOCKER IMAGE') {
            steps {
                echo 'Docker Image'

                sh '''
                    docker build -t vamshi567/wipro-project:${BUILD_NUMBER} .
                '''
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([
                        string(
                            credentialsId: 'dockerhub',
                            variable: 'dockerhub'
                        )
                    ]) {

                        sh '''
                            docker login -u vamshi567 -p ${dockerhub}

                            docker push vamshi567/wipro-project:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes'

                sh '''
                    echo "Updating EKS kubeconfig..."

                    aws eks update-kubeconfig \
                        --name ekswithvamshi1 \
                        --region ap-south-1
                    kubectl apply -f deploymentfiles/deploy.yaml
                '''
            }
        }
    }
}
