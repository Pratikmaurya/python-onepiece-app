pipeline {

    agent any
    stages {
        stage("Git checkout") {
            steps {
                echo "Cloniing the Pythonn project source code from the GitHub repository" [cite: 259]
                git branch: 'main', url: 'https://github.com/Pratikmaurya/python-onepiece-app' // **Ensure this is your forked repo URL**
            }
        }

        stage("python build") { [cite: 263]
            steps {
                echo "Building the Python application" [cite: 265]
                sh '''
                rm -rf venv # Remove existing virtual environment if any
                python -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                ''' [cite: 267-271]
            }
        }
        stage("Python test") { [cite: 274]
            steps {
                echo "Running tests for the Python application" [cite: 276]
                sh '''
                . venv/bin/activate
                pytest app/tests/test_app.py
                ''' [cite: 278-280]
            }
        }
        stage("sonarQube scan") { [cite: 283]
            environment { SONARQUBE_SCANNER_HOME = tool 'sonar-scanner' } [cite: 284]
            steps {
                echo "Running SonarQube scan for code quality analysis" [cite: 286]
                withSonarQubeEnv('sonarqube-local') { [cite: 287]
                    sh '''
                    . venv/bin/activate
                    $SONARQUBE_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Jenkins \
                    -Dsonar.sources=. \
                    ''' [cite: 289-293]
                }
            }
        }
        stage("publish sonarQube quality gate") { [cite: 297]
            environment { SONARQUBE_SCANNER_HOME = tool 'sonar-scanner' } [cite: 298]
            steps {
                echo "Re-running SonarQube scan for code quality analysis" [cite: 300]
                withSonarQubeEnv('sonarqube-local') { [cite: 301]
                    sh '''
                    . venv/bin/activate
                    $SONARQUBE_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Jenkins \
                    -Dsonar.sources=. \
                    -Dsonarqualitygate.wait=false
                    ''' [cite: 303-308]
                }
            }
        }
        stage("Docker build and push") { [cite: 312]
            steps {
                echo "Building and Pushing the Docker image to Docker Hub" [cite: 314]
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) { [cite: 315]
                    sh '''
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    docker build -t kumarpm/python-onepiece-app:latest .  // **CHANGE 'kumarpm' to your Docker Hub username**
                    docker push kumarpm/python-onepiece-app:latest  // **CHANGE 'kumarpm' to your Docker Hub username**
                    ''' [cite: 317-320]
                }
            }
        }
        stage("Trivy security scan") { [cite: 324]
            steps {
                echo "Performing security scan using Trivy" [cite: 326]
                sh '''
                trivy image --format table --severity HIGH,CRITICAL \
                --output trivy-report.txt kumarpm/python-onepiece-app:latest // **CHANGE 'kumarpm' to your Docker Hub username**
                ''' [cite: 328-330]
            }
            post { [cite: 332]
                always { [cite: 333]
                    archiveArtifacts artifacts: 'trivy-report.txt' [cite: 334]
                }
            }
        }
    }
}
