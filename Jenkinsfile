pipeline {
    agent any

    stages {
        stage ('Get Code') {
            steps {
                git 'https://github.com/Minibomba2/helloworld.git'
            }
        }

        stage ('Unit'){
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    py -m coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                    py -m coverage xml
                '''
                junit 'result-unit.xml'
            }
        }

        stage('Rest') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set FLASK_APP=app\\api.py
                        start flask run
                        start java -jar C:\\Users\\USER\\Documents\\UNIR\\wiremock\\wiremock-jre8-standalone-2.28.0.jar --port 9090 -v --root-dir C:\\Users\\USER\\Documents\\UNIR\\wiremock

                        ping -n 10 127.0.0.1

                        pytest --junitxml=result-rest.xml test\\rest
                    '''
                }
                junit 'result-rest.xml'
            }
        }

        stage ('Static') {
            steps {
                bat '''
                    flake8 --exit-zero --format=pylint app >flake8.out
                '''
                recordIssues(
                    tools: [flake8(name: 'flake8', pattern: 'flake8.out')],
                    qualityGates: [
                        [threshold: 8,  type: 'TOTAL', unstable: true],
                        [threshold: 10, type: 'TOTAL', unstable: false]
                    ]
                )
            }
        }

        stage ('Security') {
            steps {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues(
                    tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],
                    qualityGates: [
                        [threshold: 2, type: 'TOTAL', unstable: true],
                        [threshold: 4, type: 'TOTAL', unstable: false]
                    ]
                )
            }
        }

        stage ('Coverage') {
            steps {
                recordCoverage(
                    tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']],
                    qualityGates: [
                        [metric: 'LINE',   threshold: 85.0, criticality: 'ERROR'],
                        [metric: 'LINE',   threshold: 95.0, criticality: 'UNSTABLE'],
                        [metric: 'BRANCH', threshold: 80.0, criticality: 'ERROR'],
                        [metric: 'BRANCH', threshold: 90.0, criticality: 'UNSTABLE']
                    ]
                )
            }
        }

        stage ('Performance') {
            steps {
                bat '''
                    set FLASK_APP=app\\api.py
                    start flask run
                    ping -n 4 127.0.0.1
                    C:\\Users\\USER\\Documents\\UNIR\\Jmeter\\apache-jmeter-5.6.3\\bin\\jmeter.bat -n -t test\\jmeter\\flask.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}
