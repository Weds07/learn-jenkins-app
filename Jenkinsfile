pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '42dd4a42-af36-4c76-80fe-2ee8c85ccffe'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            steps {
                echo "🔧 Checking required files..."
                sh '''
                    test -f index.html || (echo "❌ Missing index.html" && exit 1)
                    test -f netlify/functions/quote.js || (echo "❌ Missing quote function" && exit 1)
                    echo "✅ Build check passed."
                    npm install
                '''
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Testing quote function load..."
                sh '''
                    node -e "require('./netlify/functions/quote.js'); console.log('✅ Function loaded successfully')"
                '''
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'netlify/cli:latest' 
                    reuseNode true
                }
            }
            steps {
                echo "🚀 Deploying to Netlify..."
                sh '''
                    netlify deploy \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID \
                      --dir=. \
                      --prod
                '''
            }
        }

        stage('Post Deploy') {
            steps {
                echo "✅ Deployment complete! Your app is live."
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD pipeline finished successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
