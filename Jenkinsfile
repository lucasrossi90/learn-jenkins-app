pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'b6d1601a-f52c-4768-814b-bf558ba8f0f3'
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_SECRET')
    }

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

        /*stage('Run tests'){
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
                 
                    post {
                        always {
                            junit 'test2-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML E2E local report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }*/

        stage('Deploy STG') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy_output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy_output.json
                '''
            
                script {
                    env.STG_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy_output.json", returnStdout: true)
                }
                
                echo "STG_URL: ${env.STG_URL}"

            }

        }

        stage('E2E STG dynamic'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.44.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.STG_URL}"
            }

            steps {
                sh "echo ${env.STG_URL}"
                sh '''
                    npx playwright install chromium
                    npx playwright test --reporter=html
                '''
            }
        
            post {
                always {
                    junit 'test2-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML E2E STG report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                  input 'Approve?'
                }
            }
        }

        stage('Deploy PROD') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('E2E Prod'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.44.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://inspiring-llama-3b0c6f.netlify.app'
            }

            steps { 
                sh '''
                    npx playwright install chromium
                    npx playwright test --reporter=html
                '''
            }
        
            post {
                always {
                    junit 'test2-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML E2E Prod report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
  
}
