pipeline {
    agent any
    stages {
        stage('Clone Repo') {
            steps {
                script {
                    logEnvironment()
                }
                git url: 'https://github.com/cedriclecalvez/helloworld-devops-unir'
            }
        }
      
      
        stage('Run Tests') {
            parallel{
                stage('Start Flask API') {
                    steps {
                        catchError(buildResult:'SUCCESS',stageResult:'ABORTED'){
                            timeout(time: 60, unit: 'SECONDS') {
                            echo 'Starting Flask service in background...'
                                bat '''
                                    set FLASK_APP=app\\api.py
                                    set PYTHONPATH=.
                                    start /B python -m flask run --host=0.0.0.0 --port=5000
                                '''
                            }
                        }
                    }
                }
                stage('Unit Tests') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest --junitxml=result-unit.xml test/unit
                                coverage xml
                            '''
                            junit 'result-unit.xml'
                        }
                    }
                }
                stage('Test Coverage') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,90,80', lineCoverageTargets: '100,95,85', onlyStable: false
                        }
                    }
                }
                stage('Static Analysis') {
                    steps {
                        bat '''
                            python -m flake8 --exit-zero --format=pylint app >flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                    }
                } 
                stage('Security Analysis') {
                    steps {
                        bat '''
                            bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}]: {msg}"
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                    }
                } 
                stage('Performance Tests') {
                    steps {
                        script{
                                logEnvironment()
                                echo 'Starting JMeter...'
                                    bat '''
                                        cd "C:\\Program Files\\apache-jmeter-5.6.3\\bin"
                                        jmeter -n -t "%WORKSPACE%\\test\\jmeter\\flask.jmx" -f -l "%WORKSPACE%\\flask.jtl"
                                    '''
                                echo 'JMeter execution finished.'
                        }
                       perfReport sourceDataFiles: 'flask.jtl'
                    }
                } 
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/*.xml, **/*.out,**/*.jtl', fingerprint: true
            echo 'Artifacts archived.'
        }
        success {
            cleanWs()
            echo 'Cleaning done'
            echo 'Pipeline completed successfully!'
        }
        failure {
            cleanWs()
            echo 'Cleaning done'
            echo 'Pipeline failed.'
        }
    }
}
def logEnvironment() {
    bat 'whoami'
    bat 'hostname'
    bat 'echo %WORKSPACE%'
    bat 'dir'
}