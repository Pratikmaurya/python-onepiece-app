pipeline {
    agent any
    stages {
        stage("Git checkout") {
            steps {
                echo "Cloning the Python project source code from the GitHub repository"
                // Make sure this URL is your forked repo!
                git branch: 'main', url: 'https://github.com/Pratikmaurya/python-onepiece-app.git'
            }
        }

        stage("python build") {
            steps {
                echo "Building the Python application"
                sh '''
                rm -rf venv
                python -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }
        stage("Python test") {
            steps {
                echo "Running tests for the Python application"
                sh '''
                . venv/bin/activate
                pytest app/tests/test_app.py
                '''
            }
        }
        stage("sonarQube scan") {
            environment { SONARQUBE_SCANNER_HOME = tool 'sonar-scanner' }
            steps {
                echo "Running SonarQube scan for code quality analysis"
                withSonarQubeEnv('sonarqube-local') {
                    sh '''
                    . venv/bin/activate
                    $SONARQUBE_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Jenkins \
                    -Dsonar.sources=. \
                    '''
                }
            }
        }
        stage("publish sonarQube quality gate") {
            environment { SONARQUBE_SCANNER_HOME = tool 'sonar-scanner' }
            steps {
                echo "Checking the SonarQube Quality Gate"
                withSonarQubeEnv('sonarqube-local') {
                    sh '''
                    . venv/bin/activate
                    $SONARQUBE_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Jenkins \
                    -Dsonar.sources=. \
                    -Dsonarqualitygate.wait=true
                    '''
                }
            }
        }
        stage("Docker build and push") {
            steps {
                echo "Building and Pushing the Docker image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh '''
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    
                    // !!! IMPORTANT: CHANGE 'kumarpm' to YOUR Docker Hub username !!!
                    docker build -t kumarpm/python-onepiece-app:latest .
                    docker push kumarpm/python-onepiece-app:latest
                    '''
                }
            }
        }
        stage("Trivy security scan") {
            steps {
                echo "Performing security scan using Trivy"
                sh '''
                // !!! IMPORTANT: CHANGE 'kumarpm' to YOUR Docker Hub username !!!
                trivy image --format table --severity HIGH,CRITICAL \
                --output trivy-report.txt kumarpm/python-onepiece-app:latest
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt'
                }
            }
        }
    }
}
