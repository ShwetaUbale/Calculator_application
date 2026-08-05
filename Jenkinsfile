pipeline {
    agent any

    stages {
        stage('Install dependencies') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e
                            PYTHON=$(command -v python3 || command -v python || true)

                            if [ -z "$PYTHON" ]; then
                                echo "No Python executable found"
                                exit 1
                            fi

                            # Check if venv works; if not, try installing python3-venv/pip via apt
                            if ! "$PYTHON" -m venv .venv >/dev/null 2>&1; then
                                echo "venv module missing or broken. Attempting apt install..."
                                (apt-get update && apt-get install -y python3-venv python3-pip) || \
                                (sudo apt-get update && sudo apt-get install -y python3-venv python3-pip)
                            fi

                            # Create virtual environment
                            "$PYTHON" -m venv .venv

                            # Upgrade pip and install dependencies
                            .venv/bin/python -m pip install --upgrade pip
                            .venv/bin/python -m pip install -r requirements.txt
                        '''
                    } else {
                        bat 'python -m venv .venv'
                        bat '.venv\\Scripts\\python.exe -m pip install --upgrade pip'
                        bat '.venv\\Scripts\\python.exe -m pip install -r requirements.txt'
                    }
                }
            }
        }

        stage('Run application') {
            steps {
                script {
                    if (isUnix()) {
                        sh '.venv/bin/python app.py'
                    } else {
                        bat '.venv\\Scripts\\python.exe app.py'
                    }
                }
            }
        }

        stage('Run tests') {
            steps {
                script {
                    if (isUnix()) {
                        sh '.venv/bin/python -m coverage run -m pytest --junitxml=pytest-results.xml'
                    } else {
                        bat '.venv\\Scripts\\python.exe -m coverage run -m pytest --junitxml=pytest-results.xml'
                    }
                }
            }
        }

        stage('Generate coverage report') {
            steps {
                script {
                    if (isUnix()) {
                        sh '.venv/bin/python -m coverage xml'
                    } else {
                        bat '.venv\\Scripts\\python.exe -m coverage xml'
                    }
                }
            }
        }

        stage('Run lint') {
            steps {
                script {
                    if (isUnix()) {
                        sh '.venv/bin/python -m flake8 .'
                    } else {
                        bat '.venv\\Scripts\\python.exe -m flake8 .'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'coverage.xml, pytest-results.xml', allowEmptyArchive: true
        }
        success {
            echo 'Build, tests, linting, and coverage report completed successfully!'
        }
        failure {
            echo 'Pipeline build failed. Check stage logs above for details.'
        }
    }
}