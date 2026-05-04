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

        stage('Build Docker Image') {
            steps {
                echo 'Construction de l image Docker...'
                bat 'docker build -t achat-app .'
            }
        }

        stage('Run Docker Compose') {
            steps {
                echo 'Demarrage du stack Docker...'
                bat '''
                docker compose down -v || exit 0
                docker compose up -d
                docker ps
                '''
            }
        }

        stage('Publish to Nexus') {
            steps {
                echo 'Publication de l artifact dans Nexus...'
                configFileProvider([configFile(fileId: 'nexus-settings', variable: 'MAVEN_SETTINGS')]) {
                    bat 'mvn deploy -s %MAVEN_SETTINGS% -DskipTests'
                }
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



while ($true) {
    $endpoints = @(
        "http://localhost:8089/SpringMVC/categorieProduit/retrieve-all-categorieProduit",
        "http://localhost:8089/SpringMVC/produit/retrieve-all-produits",
        "http://localhost:8089/SpringMVC/fournisseur/retrieve-all-fournisseurs",
        "http://localhost:8089/SpringMVC/stock/retrieve-all-stocks",
        "http://localhost:8089/SpringMVC/secteurActivite/retrieve-all-secteurActivite",
        "http://localhost:8089/SpringMVC/operateur/retrieve-all-operateurs",
        "http://localhost:8089/SpringMVC/facture/retrieve-all-factures",
        "http://localhost:8089/SpringMVC/reglement/retrieve-all-reglements",
        "http://localhost:8089/SpringMVC/actuator/health"
    )
    $url = $endpoints | Get-Random
    try {
        Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 5 | Out-Null
        Write-Host "OK  - $url"
    } catch {
        Write-Host "ERR - $url"
    }
    Start-Sleep -Milliseconds 500
}
