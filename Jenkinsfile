pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker{
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
        stage('test'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    echo 'at test stage'
                    test -f build/index.html
                    npm test
                    '''
            }
        }

        stage('E2E tests'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    echo 'starting the server'
                    npm install -g serve
                    node_modules/.bin/serve -s build &
                    npx playwright test
                    '''
            }
        }
    }
    post{
        always{
            junit 'jest-results/junit.xml'
        }
    }
}
