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
                    echo "Building WebGoat"
                    echo "Dependencies are routed through Nexus"
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
                    echo "Files that will be published"
                    echo "======================================"

                    ls -lh target/webgoat-2026.2-SNAPSHOT.jar
                    ls -lh pom.xml
                '''
            }
        }

        stage('Publish to Nexus Repository') {
            steps {
                echo 'Publishing WebGoat SNAPSHOT to Nexus'

                nexusPublisher(
                    nexusInstanceId: 'nxrm3',
                    nexusRepositoryId: 'maven-snapshots',

                    packages: [[
                        $class: 'MavenPackage',

                        mavenAssetList: [
                            [
                                classifier: '',
                                extension: 'jar',
                                filePath: 'target/webgoat-2026.2-SNAPSHOT.jar'
                            ],
                            [
                                classifier: '',
                                extension: 'pom',
                                filePath: 'pom.xml'
                            ]
                        ],

                        mavenCoordinate: [
                            groupId: 'org.owasp.webgoat',
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
            echo '======================================'
            echo 'WEBGOAT SUPPLY CHAIN PIPELINE SUCCESS'
            echo '======================================'
            echo 'WebGoat build: SUCCESS'
            echo 'Repository Firewall path: USED'
            echo 'CycloneDX SBOM: GENERATED'
            echo 'Lifecycle evaluation: COMPLETED'
            echo 'Nexus publication: COMPLETED'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'WEBGOAT PIPELINE FAILED'
            echo 'Check the failed stage above'
            echo '======================================'
        }

        always {
            echo "Jenkins Build Number: ${BUILD_NUMBER}"
        }
    }
}
```
