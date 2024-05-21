pipeline {
    agent any

    stages {        
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version

                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run tests'){
            parallel{
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps { 
                    echo 'Test stage'
                    sh 'test -f build/index.html'
                    sh 'npm test'
                    }
                }
            
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.44.0-jammy'
                            reuseNode true
                        }
                    }

                    steps { 
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright install chromium
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }

         stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    netlify --version
                '''
            }
        }

    }
    post {
        always {
            junit 'test2-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
