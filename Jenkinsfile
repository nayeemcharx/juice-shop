pipeline {
    agent any

    stages {
        stage('Verify SonarScanner Java Version') {
            steps {
                sh '/opt/sonar-scanner/bin/sonar-scanner --version'
            }
        }
        stage('Clone Repository') {
            steps {
                git 'https://github.com/nayeemcharx/juice-shop'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('juiceShopServer') {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=DVWA \
                            -Dsonar.sources=. \
                            -X
                    '''
                }
            }
        }
        stage('Export SonarQube Issues') {
            steps {
                withSonarQubeEnv('juiceShopServer') {
                    sh '''
                        curl -u "$SONAR_AUTH_TOKEN:" \
                        "$SONAR_HOST_URL/api/issues/search?componentKeys=DVWA&resolved=false&ps=500" \
                        -o sonarqube_issues.json
                    '''
                }
            }
        }
        stage('Import to DefectDojo') {
            steps {
                withCredentials([string(credentialsId: 'defectdojo-api-token', variable: 'DEFECTDOJO_API_TOKEN')]) {
                    script {
                        def defectdojoServer = 'http://103.217.110.2'
                        def engagementId = '10'

                        // Import SonarQube issues into DefectDojo via API
                        sh """
                            curl -X POST -H "Authorization: Token $DEFECTDOJO_API_TOKEN" \
                            -F 'file=@sonarqube_issues.json' \
                            -F 'scan_type=SonarQube API Import' \
                            -F 'engagement=${engagementId}' \
                            '${defectdojoServer}/api/v2/import-scan/'
                        """
                    }
                }
            }
        }
    }
}
