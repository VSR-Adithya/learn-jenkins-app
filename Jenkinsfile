pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0ca1baaf-19d7-4ee6-8559-bf53794c4341'
        NETLIFY_AUTH_TOKEN = credentials('netlify-personal-access-token') // Credentials should be added in Jenkins.
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
                    junit 'test-results/junit.xml' // Adds a Test results trend to Jenkins
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
                    node_modules/.bin/netlify deploy --site $NETLIFY_SITE_ID \
                     --auth $NETLIFY_AUTH_TOKEN --prod --dir build
                '''
                /*
                1. Cannot install globally with -g flag as a non root user.
                2. You get Netlify site ID after deploying a sample project.
                3. The auth Token and Site ID are provide from environment variables mentioned at the top.
                4. The Netlify CLI is used to deploy the build directory to the specified site.
                5. The --prod flag indicates that this is a production deployment.
                
                */
            }
            post {
                failure {
                    sh 'echo "\033[0;31mDeployment failed!\033[0m"'
                }
                success {
                    sh 'echo "\033[0;32mDeployment succeeded!\033[0m"'
                }
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
            sh 'echo "\033[0;32mPipeline completed successfully!\033[0m"'
        }
        failure {
            sh 'echo "\033[0;31mPipeline failed!\033[0m"'
        }
    }
}
