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

/*        stage('Deploy STG') {
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
*/
        stage('Deploy + E2E STG dynamic'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.44.0-jammy'
                    reuseNode true
                }
            }

            
            environment {
                CI_ENVIRONMENT_URL = 'test'
            }

            steps {
                sh '''
                    npx playwright install chromium
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy_output.json
                    CI_ENVIRONMENT_URL="$(node_modules/.bin/node-jq -r '.deploy_url' deploy_output.json)"
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

        stage('Deploy + E2E Prod'){
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
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
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
