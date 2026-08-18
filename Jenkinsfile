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
                    echo "Checking Java"
                    java -version

                    echo "Building WebGoat"
                    chmod +x mvnw

                    ./mvnw -B clean package -DskipTests -DskipITs
                '''
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar',
                                 fingerprint: true
            }
        }

        stage('Generate SBOM') {
            steps {
                sh '''
                    echo "Generating CycloneDX SBOM"

                    ./mvnw org.cyclonedx:cyclonedx-maven-plugin:2.8.2:makeAggregateBom \
                      -DoutputName=webgoat-sbom
                '''
            }
        }

        stage('Archive SBOM') {
            steps {
                archiveArtifacts artifacts: '**/target/webgoat-sbom.*',
                                 fingerprint: true
            }
        }

        stage('Sonatype Lifecycle Scan') {
            steps {
                script {
                    nexusPolicyEvaluation(
                        failBuildOnNetworkError: true,
                        iqApplication: selectedApplication('webgoat'),
                        iqScanPatterns: [[scanPattern: '**/target/*.jar']],
                        iqStage: 'build'
                    )
                }
            }
        }
    }

    post {
        success {
            echo 'WebGoat pipeline completed successfully'
        }

        failure {
            echo 'WebGoat pipeline failed'
        }
    }
}
