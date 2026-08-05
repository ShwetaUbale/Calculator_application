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

                            # Ensure venv is created
                            "$PYTHON" -m venv .venv || { echo "venv module missing. Install python3-venv on agent."; exit 1; }

                            # Upgrade pip and install dependencies inside virtual environment
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
    }
}