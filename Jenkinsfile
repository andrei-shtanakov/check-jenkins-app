pipeline {
    agent any

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
        
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh 'npm test'
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
                    
                    # Проверка наличия папки build
                    if [ ! -d "build" ]; then
                        echo "ERROR: build directory not found!"
                        exit 1
                    fi
                    
                    # Проверка наличия index.html
                    if [ ! -f "build/index.html" ]; then
                        echo "ERROR: build/index.html not found!"
                        exit 1
                    fi
                    
                    # Проверка размера файла (не пустой)
                    if [ ! -s "build/index.html" ]; then
                        echo "ERROR: build/index.html is empty!"
                        exit 1
                    fi
                    
                    echo "✅ All build artifacts are present"
                    ls -la build/
                '''
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
        }
    }
    post {
        always {
            junit 'jest-results/junit.xml'
	    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
