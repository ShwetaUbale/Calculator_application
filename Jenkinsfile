pipeline {
    agent any

    stage('Install dependencies') {
    steps {
        script {
            if (isUnix()) {
                // Ensure pip is installed first, then install requirements
                sh '''
                    python3 -m ensurepip --upgrade || sudo apt-get update && sudo apt-get install -y python3-pip
                    python3 -m pip install --upgrade pip
                    python3 -m pip install -r requirements.txt
                '''
            } else {
                bat 'python -m pip install --upgrade pip'
                bat 'python -m pip install -r requirements.txt'
            }
        }
    }
}

        stage('Run tests') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'python3 -m coverage run -m pytest --junitxml=pytest-results.xml'
                    } else {
                        bat 'python -m coverage run -m pytest --junitxml=pytest-results.xml'
                    }
                }
            }
        }

        stage('Generate coverage report') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'python3 -m coverage xml'
                    } else {
                        bat 'python -m coverage xml'
                    }
                }
            }
        }

        stage('Run lint') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'flake8 .'
                    } else {
                        bat 'flake8 .'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'coverage.xml, pytest-results.xml', allowEmptyArchive: true
        }
    }
}
