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

                            if ! "$PYTHON" -m pip --version >/dev/null 2>&1; then
                                TMPFILE=$(mktemp /tmp/get-pip.XXXXXX.py)
                                "$PYTHON" - <<'PY'
import urllib.request, pathlib, sys
url = 'https://bootstrap.pypa.io/get-pip.py'
path = pathlib.Path(sys.argv[1])
path.write_bytes(urllib.request.urlopen(url).read())
PY
 "$TMPFILE"
                                "$PYTHON" "$TMPFILE" --user
                            fi

                            "$PYTHON" -m pip install --user --upgrade pip
                            "$PYTHON" -m pip install --user -r requirements.txt
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
                        sh '''
                            PYTHON=$(command -v python3 || command -v python || true)
                            if [ -z "$PYTHON" ]; then
                                echo "No Python executable found"
                                exit 1
                            fi
                            "$PYTHON" -m coverage run -m pytest --junitxml=pytest-results.xml
                        '''
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
                        sh '''
                            PYTHON=$(command -v python3 || command -v python || true)
                            if [ -z "$PYTHON" ]; then
                                echo "No Python executable found"
                                exit 1
                            fi
                            "$PYTHON" -m coverage xml
                        '''
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
                        sh '''
                            PYTHON=$(command -v python3 || command -v python || true)
                            if [ -z "$PYTHON" ]; then
                                echo "No Python executable found"
                                exit 1
                            fi
                            "$PYTHON" -m flake8 .
                        '''
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
