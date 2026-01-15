pipeline {
    agent any
    
    environment {
        // Staging Server Details
        STAGING_SERVER = '103.142.69.163'
        STAGING_PORT = '4345'
        STAGING_USER = 'misl'
        
        // Project Details
        APP_NAME = 'tourism-website'
        DEPLOY_PATH = '/home/misl/tourism-website'
        GIT_REPO = 'https://github.com/imNHK07/Test.git'  // Update this with your repository URL
        GIT_BRANCH = 'main'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=== Checking out code ==='
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    userRemoteConfigs: [[url: "${GIT_REPO}"]]
                ])
            }
        }
        
        stage('Build') {
            steps {
                echo '=== Building application ==='
                sh '''
                    echo "Installing dependencies..."
                    # If you have npm/yarn dependencies
                    # npm install
                    # npm run build
                    
                    echo "Build completed successfully"
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo '=== Running tests ==='
                sh '''
                    echo "Running test suite..."
                    # npm test
                    echo "Tests completed"
                '''
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                echo '=== Deploying to Staging Server ==='
                sshagent(['staging-ssh-credentials']) {
                    sh '''
                        # Create deployment directory if it doesn't exist
                        ssh -o StrictHostKeyChecking=no \
                            -p ${STAGING_PORT} \
                            ${STAGING_USER}@${STAGING_SERVER} \
                            "mkdir -p ${DEPLOY_PATH}"
                        
                        # Copy files to staging server
                        scp -o StrictHostKeyChecking=no \
                            -P ${STAGING_PORT} \
                            -r ./* \
                            ${STAGING_USER}@${STAGING_SERVER}:${DEPLOY_PATH}/
                        
                        # Set proper permissions
                        ssh -o StrictHostKeyChecking=no \
                            -p ${STAGING_PORT} \
                            ${STAGING_USER}@${STAGING_SERVER} \
                            "cd ${DEPLOY_PATH} && chmod -R 755 . && ls -la"
                    '''
                }
            }
        }
        
        stage('Restart Services') {
            steps {
                echo '=== Restarting services on Staging Server ==='
                sshagent(['staging-ssh-credentials']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -p ${STAGING_PORT} \
                            ${STAGING_USER}@${STAGING_SERVER} \
                            "cd ${DEPLOY_PATH} && \
                            sudo systemctl restart nginx && \
                            sudo systemctl status nginx"
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo '=== Performing health check ==='
                sh '''
                    sleep 5
                    echo "Checking server health..."
                    # Add your health check logic here
                    # curl -f http://103.142.69.163/health || exit 1
                    echo "Health check passed"
                '''
            }
        }
    }
    
    post {
        success {
            echo '=== Deployment Successful ==='
            // Add Slack/Email notification here
            // slackSend(channel: '#deployments', message: "Staging deployment successful: ${BUILD_URL}")
        }
        
        failure {
            echo '=== Deployment Failed ==='
            // Add Slack/Email notification here
            // slackSend(channel: '#deployments', color: 'danger', message: "Staging deployment failed: ${BUILD_URL}")
        }
        
        always {
            cleanWs()
        }
    }
}
