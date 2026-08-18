pipeline {
    agent any

    environment {
        JAVA_HOME = '/opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Build via Nexus Firewall') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Checking Java"
                    echo "======================================"

                    java -version

                    echo "======================================"
                    echo "Building WebGoat through Nexus"
                    echo "======================================"

                    chmod +x mvnw

                    ./mvnw -B clean package \
                      -DskipTests \
                      -DskipITs
                '''
            }
        }

        stage('Archive JAR') {
            steps {
                echo 'Archiving WebGoat JAR'

                archiveArtifacts(
                    artifacts: 'target/webgoat-2026.2-SNAPSHOT.jar',
                    fingerprint: true
                )
            }
        }

        stage('Generate SBOM') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Generating CycloneDX SBOM"
                    echo "======================================"

                    ./mvnw \
                      org.cyclonedx:cyclonedx-maven-plugin:2.8.2:makeAggregateBom \
                      -DoutputName=webgoat-sbom
                '''
            }
        }

        stage('Archive SBOM') {
            steps {
                echo 'Archiving CycloneDX SBOM'

                archiveArtifacts(
                    artifacts: 'target/webgoat-sbom.*',
                    fingerprint: true
                )
            }
        }

        stage('Sonatype Lifecycle Scan') {
            steps {
                script {
                    echo 'Starting Sonatype Lifecycle evaluation'

                    nexusPolicyEvaluation(
                        failBuildOnNetworkError: true,
                        iqApplication: selectedApplication('webgoat'),
                        iqScanPatterns: [[
                            scanPattern: 'target/webgoat-2026.2-SNAPSHOT.jar'
                        ]],
                        iqStage: 'build'
                    )
                }
            }
        }

        stage('Verify Publish Files') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Checking files before Nexus publish"
                    echo "======================================"

                    ls -lh target/webgoat-2026.2-SNAPSHOT.jar
                    ls -lh pom.xml
                    ls -lh target/webgoat-sbom.json
                '''
            }
        }

        stage('Publish to Nexus Repository') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Publishing WebGoat SNAPSHOT to Nexus"
                    echo "======================================"

                    ./mvnw deploy:deploy-file \
                      -DgroupId=org.owasp.webgoat \
                      -DartifactId=webgoat \
                      -Dversion=2026.2-SNAPSHOT \
                      -Dpackaging=jar \
                      -Dfile=target/webgoat-2026.2-SNAPSHOT.jar \
                      -DpomFile=pom.xml \
                      -DrepositoryId=nexus-snapshots \
                      -Durl=http://localhost:8081/repository/maven-snapshots/
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'WEBGOAT SUPPLY CHAIN PIPELINE SUCCESS'
            echo '======================================'
            echo 'Build: SUCCESS'
            echo 'Repository Firewall path: USED'
            echo 'SBOM: GENERATED'
            echo 'Lifecycle Scan: COMPLETED'
            echo 'Artifact: PUBLISHED TO NEXUS'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'WEBGOAT SUPPLY CHAIN PIPELINE FAILED'
            echo 'Check the failed Jenkins stage above'
            echo '======================================'
        }

        always {
            echo "Jenkins Build Number: ${BUILD_NUMBER}"
        }
    }
}
