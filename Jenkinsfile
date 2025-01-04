pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                // Lancement des tests unitaires.
                bat "./gradlew test"
            }
            post {
                always {
                    // Archivage des résultats des tests unitaires.
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        stage('SonarQube analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        // Exécution de l'analyse SonarQube.
                        bat "./gradlew sonar"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Vérification de la qualité du code.
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    }
}
