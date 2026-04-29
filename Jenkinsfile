pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Recuperation du code source depuis GitHub...'
                checkout scm
            }
        }

        stage('Clean & Compile') {
            steps {
                echo 'Nettoyage et compilation du projet...'
                bat 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Analyse de la qualite du code avec SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    bat 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=achat -Dsonar.projectName=achat'
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Generation du fichier JAR...'
                bat 'mvn package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archivage du fichier JAR genere...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Pipeline execute avec succes.'
        }
        failure {
            echo 'Le pipeline a echoue.'
        }
    }
}