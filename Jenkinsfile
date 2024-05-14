pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                sh '''
                ls -la
                touch no-docker.txt
                '''
                echo 'Hello World'
            }
        }
        
        stage('Docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Hello Docker'
                sh '''
                    ls -la
                    touch docker.txt
                '''
            }
        }
    }
}
