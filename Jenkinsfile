pipeline {
    agent any

    triggers {
       // cron('* * * * *')   // every minute (optional)
    }

    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    stages {

        stage('Clone Code from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/durganaresh83/CapStone-B13-shopNow.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    cd frontend && npm install || true
                    cd ../backend && npm install || true
                    cd ../admin && npm install || true
                '''
            }
        }

        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        $SONAR_SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=shopNow \
                        -Dsonar.projectKey=shopNow \
                        -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '''
                    --scan ./
                    --format XML
                    --disableYarnAudit
                    --failOnCVSS 7
                ''',
                odcInstallation: 'OWASP'

                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonar Quality Gate Scan') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            currentBuild.result = 'UNSTABLE'
                        }
                    }
                }
            }
        }

        stage('Trivy Scan + Metrics') {
            steps {
                retry(3) {
                    sh '''
                        docker run --rm \
                          -v $(pwd):/project \
                          aquasec/trivy fs \
                          --format json \
                          --output trivy.json \
                          /project

                        CRITICAL=$(jq '[.Results[].Vulnerabilities[]? | select(.Severity=="CRITICAL")] | length' trivy.json)
                        HIGH=$(jq '[.Results[].Vulnerabilities[]? | select(.Severity=="HIGH")] | length' trivy.json)
                        MEDIUM=$(jq '[.Results[].Vulnerabilities[]? | select(.Severity=="MEDIUM")] | length' trivy.json)

                        cat <<EOF > metrics.prom
vulnerability_critical $CRITICAL
vulnerability_high $HIGH
vulnerability_medium $MEDIUM
EOF

                        curl -X POST http://<EC2-IP>:9091/metrics/job/trivy_scan \
                          --data-binary @metrics.prom
                    '''
                }
            }
        }

        stage('Trivy File System Scan (Gate)') {
            steps {
                sh '''
                    trivy fs . \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      --format table
                '''
            }
        }

        stage('Deploy using Docker Compose') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }

    post {

        always {
            slackSend(
                color: '#439FE0',
                message: """🔄 *Build Update*
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}
URL: ${env.BUILD_URL}
"""
            )
        }

        success {
            slackSend(
                color: 'good',
                message: """✅ *BUILD SUCCESS*
Deployment completed successfully 🚀
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
"""
            )
        }

        unstable {
            slackSend(
                color: 'warning',
                message: """⚠️ *BUILD UNSTABLE*
Quality Gate failed
Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
"""
            )
        }

        failure {
            slackSend(
                color: 'danger',
                message: """🚨 *BUILD FAILED*
Critical vulnerabilities detected ❌
Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
Check logs immediately!
"""
            )
        }
    }
}
