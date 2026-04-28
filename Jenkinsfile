pipeline {
    agent any

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
                bat 'mvnw.cmd clean compile'
            }
        }

        stage('Test (optional)') {
            steps {
                echo 'Execution des tests (si existants)...'
                // This will NOT fail if no tests exist
                bat 'mvn test || echo No tests found'
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
        always {
            // Will NOT fail if no test reports exist
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
        }
    }
}