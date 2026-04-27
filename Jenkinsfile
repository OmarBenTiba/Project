pipeline {
    agent any

    tools {
        maven 'Maven' // must match Jenkins config
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/OmarBenTiba/Project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'sqp_0ea3dcf14808be4a06c88ad45e20e81d933235a1')]) {
                        sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=achat \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                        """
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

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
