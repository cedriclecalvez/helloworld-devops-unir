pipeline {
    agent any
    environment {
        VIRTUAL_ENV = '.venv'
    }
    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'dev', url: 'https://github.com/cedriclecalvez/helloworld-devops-unir'
            }
        }
        stage('Build') {
            steps {
                echo 'NO HAY QUE COMPILAR. ESTO ES PYTHON'
                bat "dir"
            }
        }
        stage('Setup Virtual Environment') {
            steps {
                bat 'python -m venv ${VIRTUAL_ENV}'
            }
        }
        stage('Install Dependencies') {
            steps {
                bat 'call ${VIRTUAL_ENV}\\Scripts\\activate.bat && python -m pip install --upgrade pip && pip install pytest Flask'
            }
        }
      

        stage('Tests'){
            parallel{
                stage('Start Flask API') {
                    steps {
                        timeout(time: 50, unit: 'SECONDS') {
                            echo 'Starting Flask service in background...'
                            bat 'set FLASK_APP=app\\api.py && set PYTHONPATH=. && start /B call ${VIRTUAL_ENV}\\Scripts\\activate.bat && ${VIRTUAL_ENV}\\Scripts\\python -m flask run'
                        }
                    }
                }
                 stage('Start Wiremock Service') {
                    steps {
                        timeout(time: 30, unit: 'SECONDS') {
                            echo 'Starting Wiremock service in background...'
                            bat 'icacls C:\\ProgramData\\Jenkins\\wiremock-standalone-3.10.0.jar'
                            bat 'start /B java -Dfile.encoding=UTF-8 -jar C:\\ProgramData\\Jenkins\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir %WORKSPACE%\\test\\wiremock'
                            echo 'WireMock started in background'
                        }
                    }
                }
                stage('Run Unit Tests') {
                    steps {
                        catchError(buildResult:'UNSTABLE',stageResult:'FAILURE'){
                            bat 'call ${VIRTUAL_ENV}\\Scripts\\activate.bat && ${VIRTUAL_ENV}\\Scripts\\python -m pytest --junitxml=result-unit.xml test\\unit'
                            junit 'result-unit.xml'
                        }
                    }
                }
                stage('Run REST Tests') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat 'call ${VIRTUAL_ENV}\\Scripts\\activate.bat && ${VIRTUAL_ENV}\\Scripts\\python -m pytest --junitxml=result-rest.xml test\\rest'
                            junit 'result-rest.xml'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/result-*.xml', fingerprint: true
            cleanWs()
            echo 'Cleaning done and artifacts archived.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}