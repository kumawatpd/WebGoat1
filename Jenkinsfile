pipeline {
    agent any

    environment {
        JAVA_HOME = '/opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    java -version
                    chmod +x mvnw
                    ./mvnw -B clean package -DskipTests -DskipITs
                '''
            }
        }

        stage('Nexus IQ Scan') {
            steps {
                script {
                    def policyEvaluation = nexusPolicyEvaluation(
                        failBuildOnNetworkError: true,
                        iqApplication: selectedApplication('webgoat-legacy'),
                        iqScanPatterns: [[scanPattern: '**/target/*.jar']],
                        iqStage: 'build',
                        jobCredentialsId: 'admin'
                    )

                    echo "IQ Scan URL: ${policyEvaluation.applicationCompositionReportUrl}"
                }
            }
        }
    }
}
