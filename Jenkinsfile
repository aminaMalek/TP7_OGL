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

        // Analyse SonarQube.
        stage('SonarQube analysis') {
            withSonarQubeEnv('sonarqube') {
                 bat "./gradlew sonar"
            }

        }
    }
}
