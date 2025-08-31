pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '66e49d1c-22d5-43bf-b470-9b0948e07cb9'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }

        // stage('AWS') {
        //     agent {
        //         docker {
        //             image 'amazon/aws-cli'
        //             reuseNode true
        //             args "--entrypoint=''"
        //         }
        //     }
        //     environment {
        //         AWS_S3_BUCKET = 'amza-learn-jenkins'
        //     }
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        //             sh '''
        //                 aws --version
        //                 aws s3 sync build s3://$AWS_S3_BUCKET
        //             '''
        //         }
        //     }
        // }

        // stage('Tests') {
        //     parallel {
        //         stage('Unit tests') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }

        //             steps {
        //                 sh '''
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     publishHTML(target: [
        //                         reportDir: 'playwright-report',
        //                         reportFiles: 'index.html',
        //                         reportName: 'Local E2E'
        //                     ])
        //                 }
        //             }
        //         }
        //     }
        // }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify deploy --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --dir=build --json > deploy-output.json
                '''

                script {
                    def deployUrl = sh(script: "jq -r '.deploy_url' deploy-output.json", returnStdout: true).trim()
                    env.CI_ENVIRONMENT_URL = deployUrl
                    echo "Staging URL: ${deployUrl}"
                }

                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML(target: [
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E'
                    ])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify deploy --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML(target: [
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Prod E2E'
                    ])
                }
            }
        }
    }
}
