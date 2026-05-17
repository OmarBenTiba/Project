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

        stage('OWASP Dependency Check') {
            steps {
                echo 'Analyse des vulnerabilites OWASP...'
                bat 'mvn org.owasp:dependency-check-maven:9.0.9:check -DfailOnError=false'
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

        stage('Trivy Security Scan') {
    steps {
        echo 'Analyse de securite Docker avec Trivy...'
        bat '''
        trivy image --exit-code 0 --severity HIGH,CRITICAL --format json --output trivy-report.json achat-app

        powershell -Command "$json = Get-Content trivy-report.json | ConvertFrom-Json; $rows = foreach ($result in $json.Results) { foreach ($vuln in $result.Vulnerabilities) { [PSCustomObject]@{Target=$result.Target; Package=$vuln.PkgName; VulnerabilityID=$vuln.VulnerabilityID; Severity=$vuln.Severity; InstalledVersion=$vuln.InstalledVersion; FixedVersion=$vuln.FixedVersion; Title=$vuln.Title} } }; $rows | ConvertTo-Html -Title 'Trivy Security Report' -PreContent '<h1>Trivy Security Report - achat-app</h1>' | Out-File trivy-report.html"
        '''
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

        stage('OWASP ZAP Scan') {
    steps {
        echo 'Analyse de sécurité dynamique avec OWASP ZAP...'

        bat """
        docker run --rm ^
          -v %cd%:/zap/wrk/:rw ^
          ghcr.io/zaproxy/zaproxy:stable ^
          zap-baseline.py ^
          -t http://host.docker.internal:8089/SpringMVC ^
          -r zap-report.html ^
          -I
        """
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
                archiveArtifacts artifacts: 'target/dependency-check-report.html', allowEmptyArchive: true
                archiveArtifacts artifacts: 'trivy-report.html', allowEmptyArchive: true
                archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
                archiveArtifacts artifacts: 'zap-report.html', allowEmptyArchive: true
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