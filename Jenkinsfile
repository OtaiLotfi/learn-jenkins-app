pipeline {
    agent any
    
    environment{
        NETLIFY_SITE_ID = '87c36ae9-c978-40dc-9fde-2bf912b18be8'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent{
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

stage('Tests') {
    parallel {
        stage('OTAI-Unit Test') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                sh '''
                test -f build/index.html
                npm test
                '''
            }
            post{
              always{
                   junit 'test-results/junit.xml'
                }
           }
        }

stage('OTAI-E2E Tests') {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
    }

    steps {
        sh '''
        npm install serve
        node_modules/.bin/serve -s build &
        sleep 10
        npx playwright test --reporter=allure-playwright
        '''
    }

    post {
        always {
            script {
                allure([
                    results: [[path: 'allure-results']],
                    reportDir: 'allure-report',
                    reportName: 'OTAI Playwright Test Report'
                ])
            }
        }
    }
}

    }
}


    stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli 
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}