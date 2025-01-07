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
                        bat "./gradlew sonar --info"
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

    }
}
