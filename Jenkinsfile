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
                // Lancement de l'analyse SonarQube.
                bat "./gradlew sonar"
            }
        }

    }
}
