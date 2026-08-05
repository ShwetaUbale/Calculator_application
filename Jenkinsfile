pipeline {
    agent any

    stages {
        stage('Install dependencies') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            PYTHON=python3
                            if ! command -v "$PYTHON" >/dev/null 2>&1; then
                                PYTHON=python
                            fi

                            if ! "$PYTHON" -m pip --version >/dev/null 2>&1; then
                                "$PYTHON" - <<'PY'
import urllib.request, tempfile, pathlib, sys, os
url = 'https://bootstrap.pypa.io/get-pip.py'
tmp = pathlib.Path(tempfile.gettempdir()) / 'get-pip.py'
with urllib.request.urlopen(url) as r:
    tmp.write_bytes(r.read())
os.execv(sys.executable, [sys.executable, str(tmp), '--quiet'])
PY
                            fi

                            "$PYTHON" -m pip install --upgrade pip
                            "$PYTHON" -m pip install --user -r requirements.txt
                        '''
                    } else {
                        bat 'python -m pip --version || python -m ensurepip --upgrade'
                        bat 'python -m pip install --upgrade pip'
                        bat 'python -m pip install --user -r requirements.txt'
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
