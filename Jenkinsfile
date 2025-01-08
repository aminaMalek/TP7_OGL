pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                bat './gradlew test'
            }
            post {
                always {
                    echo 'Archiving unit test results...'
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    withSonarQubeEnv('sonarqube') {
                        bat './gradlew clean'
                        bat './gradlew sonar -Dsonar.login=6beafbaaff4dcaf702e37a53f1bf0b6c45bd06b4 -Dsonar.password=admin'
                    }
                }
            }
        }

        //   stage('Code Quality') {
        //     steps {
        //         timeout(time: 1, unit: 'HOURS') {
        //             script {
        //                 def qg = waitForQualityGate()
        //                 if (qg.status != 'OK') {
        //                       error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                  }
        //             }
        //         }
        //     }
        // }

        stage('Build') {
            steps {
                echo 'Building Project...'
                bat './gradlew build'

                echo 'Generating Documentation...'
                bat './gradlew javadoc'
            }
            post {
                always {
                    echo 'Archiving build artifacts...'
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
                script {
                    def SLACK_CHANNEL = '#build-status'
                    def EMAIL_TO_SUCCESS = 'la_malek@esi.dz'

                    try {
                        echo 'Sending Slack notification...'
                        slackSend(
                            channel: SLACK_CHANNEL,
                            color: 'good',
                            message: """
                                ✅ Deploy succeeded!
                                Project: ${env.JOB_NAME}
                                Build Number: ${env.BUILD_NUMBER}
                                More info: ${env.BUILD_URL}
                            """
                        )
                    } catch (Exception e) {
                        echo "Slack notification failed: ${e.getMessage()}"
                    }

                    try {
                        echo 'Sending email notification...'
                        mail(
                            subject: "✅ Build SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: """
                                The build has completed successfully!

                                Project: ${env.JOB_NAME}
                                Build Number: ${env.BUILD_NUMBER}
                                Build URL: ${env.BUILD_URL}
                            """,
                            to: EMAIL_TO_SUCCESS
                        )
                    } catch (Exception e) {
                        echo "Email notification failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                def SLACK_CHANNEL = '#build-status'
                def EMAIL_TO_FAILURE = 'la_malek@esi.dz'

                try {
                    echo 'Sending Slack notification for failure...'
                    slackSend(
                        channel: SLACK_CHANNEL,
                        color: 'danger',
                        message: """
                            ❌ Deploy failed!
                            Project: ${env.JOB_NAME}
                            Build Number: ${env.BUILD_NUMBER}
                            More info: ${env.BUILD_URL}
                        """
                    )
                } catch (Exception e) {
                    echo "Slack notification failed: ${e.getMessage()}"
                }

                try {
                    echo 'Sending email notification for failure...'
                    mail(
                        subject: "❌ Deploy FAILURE: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                        body: """
                            Deploy failed!

                            Project: ${env.JOB_NAME}
                            Build Number: ${env.BUILD_NUMBER}
                            Build URL: ${env.BUILD_URL}

                            Please check the build logs for more details.
                        """,
                        to: EMAIL_TO_FAILURE
                    )
                } catch (Exception e) {
                    echo "Email notification failed: ${e.getMessage()}"
                }
            }
        }
    }
}
