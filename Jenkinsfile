pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }
    
    environment {
        NETLIFY_SITE_ID = 'da0b64c5-659d-4916-92d2-6cac9cd4ad78'
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
            npm install netlify-cli  
            npx netlify --version   
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
