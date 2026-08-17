
pipeline {
    agent any

    environment {
        ARTEFACT_NAME = "${WORKSPACE}/target/WebGoat-${BUILD_VERSION}.war"
        DEV_REPO = 'staging-dev'
        TAG_FILE = "${WORKSPACE}/tag.json"
        IQ_SCAN_URL = ""
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -Dproject.version=$BUILD_VERSION -Dmaven.test.failure.ignore clean package'
            }
            post {
                success {
                    echo 'Now archiving...'
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage('Nexus IQ Scan'){
            steps {
                script{
                
                        def policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: true, iqApplication: selectedApplication('webgoat-legacy'), iqScanPatterns: [[scanPattern: '**/*.war']], iqStage: 'build', jobCredentialsId: 'admin'
                        echo "Nexus IQ scan succeeded: ${policyEvaluation.applicationCompositionReportUrl}"
                        IQ_SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                    
                }
            }
        }

        
    }
}


