pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '36a1f7a2-4fe5-4de7-ac29-fe9486a3f5ed'
	NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }


    stages {
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh 'npm run build'
            }
        }
	stage('Post Tests') {
	    parallel {
        	stage('Post-Build Checks') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Checking build artifacts..."
                            if [ ! -d "build" ]; then
                                echo "ERROR: build directory not found!"
                                exit 1
                            fi
                            if [ ! -f "build/index.html" ]; then
                                echo "ERROR: build/index.html not found!"
                                exit 1
                            fi
                            if [ ! -s "build/index.html" ]; then
                                echo "ERROR: build/index.html is empty!"
                                exit 1
                            fi
                            echo "✅ All build artifacts are present"
                            ls -la build/
                        '''
                    }
		    
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
	        stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "E2E test starts..."
		            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html

                        '''
                    }

                    post {
                        always {
            junit 'jest-results/junit.xml'
	                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
		    }
		}
	    }
        }
	stage('Deploy staging') {
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
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }
        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
		    echo "Deploying to production. Site_ID: $NETLIFY_SITE_ID"
		    node_modules/.bin/netlify status
                    node_modules/.bin/netlify  deploy --dir=build --prod
                '''
            }
	}
	stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://inspiring-kringle-e9591b.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}

