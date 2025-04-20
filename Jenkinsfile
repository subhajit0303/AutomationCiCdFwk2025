pipeline {
    agent none
    
    // Trigger builds on code push or every 15 minutes
    triggers {
        pollSCM('H/15 * * * *')
    }
    
    stages {
        stage('Execute Test Suites in Parallel') {
            parallel {
                // DEFAULT SMOKE TESTS
                stage('Smoke Tests') {
                    agent {
                        docker {
                            image 'seleniarm/standalone-chromium:latest' // ARM-compatible
                            args '--shm-size=2g --cpus 2 -v /dev/shm:/dev/shm'
                            label 'm1-agent'
                        }
                    }
                    steps {
                        checkout scm
                        sh 'mvn clean test' // Runs testng/smoke-tests.xml by default
                    }
                }
                
                // DRIVER REGRESSION
                stage('Driver Regression') {
                    agent {
                        docker {
                            image 'seleniarm/standalone-chromium:latest'
                            args '--shm-size=3g --cpus 3' // More resources for regression
                            label 'm1-agent'
                        }
                    }
                    steps {
                        checkout scm
                        sh 'mvn clean test -Pdriver-regression' // Activates profile
                    }
                }
                
                // RIDER REGRESSION
                stage('Rider Regression') {
                    agent {
                        docker {
                            image 'seleniarm/standalone-chromium:latest'
                            args '--shm-size=3g --cpus 3'
                            label 'm1-agent'
                        }
                    }
                    steps {
                        checkout scm
                        sh 'mvn clean test -Prider-regression' // Activates profile
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Archive test reports and screenshots
            archiveArtifacts artifacts: 'target/surefire-reports/**/*, screenshots/*.png'
            
            // Publish JUnit results
            junit 'target/surefire-reports/**/*.xml'
            
            // Clean workspace
            cleanWs()
        }
        
        failure {
            // Send email notification on failure
            emailext body: '''${SCRIPT, template="groovy-html.template"}''',
                      subject: 'FAILED: ${JOB_NAME} - Build ${BUILD_NUMBER}',
                      to: 'subhajitm61@gmail.com',
                      attachLog: true
        }
    }
}