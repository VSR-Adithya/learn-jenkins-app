pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0ca1baaf-19d7-4ee6-8559-bf53794c4341'
        NETLIFY_AUTH_TOKEN = credentials('netlify-personal-access-token')
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

        stage('Tests') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    #test -f build/index.html
                    npm test
                '''
            }
            post {
                always {
                    junit 'test-results/junit.xml'
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
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --prod --dir build
                    sh 'echo "\e[0;32mDeployment completed successfully!\e[0m"'
                '''
            }
            post {
                failure {
                    sh 'echo "\e[0;31mDeployment failed!\e[0m"'
                }
                success {
                    sh 'echo "\e[0;32mDeployment succeeded!\e[0m"'
                }
            }
        }
        post {
            always {
                script {
                    def buildUrl = "${env.BUILD_URL}"
                    def siteUrl = "https://${NETLIFY_SITE_ID}.netlify.app"
                    sh 'echo "Build URL: ${buildUrl}"'
                    sh 'echo "Site URL: ${siteUrl}"'
                }
            }
            success {
                sh 'echo "\e[0;32mPipeline completed successfully!\e[0m"'
            }
            failure {
                sh 'echo "\e[0;31mPipeline failed!\e[0m"'
            }
        }
    }
}

