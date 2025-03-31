pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'da0b64c5-659d-4916-92d2-6cac9cd4ad78'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Setup') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔧 Setting up environment..."
                sh 'npm install netlify-cli'
            }
        }

        stage('Build') {
            steps {
                echo "🔍 Verifying required files..."
                sh '''
                    test -f index.html || (echo "❌ Missing index.html" && exit 1)
                    test -f netlify/functions/quote.js || (echo "❌ Missing quote function" && exit 1)
                    echo "✅ Build check passed."
                '''
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Running function tests..."
                sh '''
                    # No need to install npm again in node:18-alpine
                    npm install
                    node -e "require('./netlify/functions/quote.js'); console.log('✅ Function loaded successfully')"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying to Netlify..."
                sh '''
                    node_modules/.bin/netlify login
                    node_modules/.bin/netlify link --id=$NETLIFY_SITE_ID
                    node_modules/.bin/netlify deploy \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID \
                      --dir=. \
                      --prod
                '''
            }
        }

        stage('Post Deploy') {
            steps {
                echo "🔍 Checking deployment status..."
                sh '''
                    node_modules/.bin/netlify status --auth=$NETLIFY_AUTH_TOKEN
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Your app is live."
        }
        failure {
            echo "❌ Deployment failed. Check the logs for details."
        }
    }
}
