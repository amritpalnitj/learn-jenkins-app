pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='8493930b-c85b-4ef6-9a79-85f6e1721e61'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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

        stage(' Running tests in Parallel'){
            parallel{
                stage('Unit Test'){
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

                stage('E2E Tests'){
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                    echo 'starting the server'
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                    '''
                    }
                }

            }
        }

        stage('Stage Deploy') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                         npm install netlify-cli@20.1.1 node-jq
                         node_modules/.bin/netlify --version
                         echo "Deploying to Prod with Site ID: ${NETLIFY_SITE_ID}"
                         node_modules/.bin/netlify status
                         node_modules/.bin/netlify deploy --dir=build --prod --json > deploy_output.json
                     '''
                script{
                    env.staging_url = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy_output.json")
                }
            }
        }

        stage('E2E Stage Tests'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL="${env.staging_url}"
                // above parameter will make change in playwright.config.js hence ensuring deployed version is tested, if not given localhost:3030 is considered
            }
            steps{
                sh '''
                   echo " Staging url is ${env.staging_url}""

                   npx playwright test
                    '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright e2e stage Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }

        }


    }
    post{
        always{
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])

        }
    }
}
