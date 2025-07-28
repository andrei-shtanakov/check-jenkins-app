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
                    image 'mcr.microsoft.com/playwright:v1.54.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "E2E test starts..."
		    #npm install serve
		    #serve -s build
                    #npx playwright test 

		    npx serve -s build -l 3000 &
                    SERVER_PID=$!
            
                    # Ждем запуска сервера
                    sleep 5
            
                    # Проверяем доступность
                    curl -f http://localhost:3000 || exit 1
            
                    # Запускаем тесты
                    npm run test:e2e
            
                    # Останавливаем сервер
                    kill $SERVER_PID 2>/dev/null || true
                '''
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
