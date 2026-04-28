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

        stage('Package') {
            steps {
                echo 'Generation du fichier JAR...'
                bat 'mvn package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}