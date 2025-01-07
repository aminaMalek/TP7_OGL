pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                // Lancement des tests unitaires.
                bat './gradlew test'
            }
            post {
                always {
                    // Archivage des résultats des tests unitaires.
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        // Exécution de l'analyse SonarQube.
                        bat './gradlew sonar --info'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        // Vérification de la qualité du code.
                        timeout(time: 1, unit: 'MINUTES') {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building Project...'
                bat './gradlew build'
                echo 'Generating Documentation...'
                bat './gradlew javadoc'
            }
            post {
                always {
                    // Archivage des artefacts générés.
                    archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                    archiveArtifacts artifacts: 'build/docs/javadoc/**/*', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Application...'
                bat './gradlew publish'
            }
        }

        stage('Notification') {
            steps {
                echo 'Sending Notifications...'
                bat './gradlew postPublishedPluginToSlack'
                bat './gradlew sendMail'
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
