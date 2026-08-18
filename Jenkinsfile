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
                    echo "Checking Java"
                    java -version

                    echo "Building WebGoat through Nexus Repository"
                    chmod +x mvnw

                    ./mvnw -B clean package -DskipTests -DskipITs
                '''
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts(
                    artifacts: '**/target/*.jar',
                    fingerprint: true
                )
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
                archiveArtifacts(
                    artifacts: '**/target/webgoat-sbom.*',
                    fingerprint: true
                )
            }
        }

        stage('Sonatype Lifecycle Scan') {
            steps {
                script {
                    nexusPolicyEvaluation(
                        failBuildOnNetworkError: true,
                        iqApplication: selectedApplication('webgoat'),
                        iqScanPatterns: [[
                            scanPattern: '**/target/*.jar'
                        ]],
                        iqStage: 'build'
                    )
                }
            }
        }

        stage('Publish to Nexus Repository') {
            steps {
                nexusPublisher(
                    nexusInstanceId: 'nxrm3',

                    // WebGoat version contains SNAPSHOT,
                    // therefore publish to maven-snapshots
                    nexusRepositoryId: 'maven-snapshots',

                    packages: [[
                        $class: 'MavenPackage',

                        mavenAssetList: [[
                            classifier: '',
                            extension: 'jar',
                            filePath: 'target/webgoat-2026.2-SNAPSHOT.jar'
                        ]],

                        mavenCoordinate: [
                            groupId: 'org.demo',
                            artifactId: 'webgoat',
                            packaging: 'jar',
                            version: '2026.2-SNAPSHOT'
                        ]
                    ]]
                )
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'WebGoat pipeline completed successfully'
            echo 'Build: SUCCESS'
            echo 'SBOM: GENERATED'
            echo 'Lifecycle: COMPLETED'
            echo 'Artifact: PUBLISHED TO NEXUS'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'WebGoat pipeline failed'
            echo 'Check the failed Jenkins stage'
            echo '========================================'
        }

        always {
            echo "Jenkins Build Number: ${BUILD_NUMBER}"
        }
    }
}
