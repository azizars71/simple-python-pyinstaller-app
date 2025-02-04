pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                    args '--privileged'
                }
            }
            steps {
                script {
                    try {
                        sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                    args '--privileged'
                }
            }
            steps {
                script {
                    try {
                        sh 'docker --version'
                        sh 'docker ps'
                        sh 'docker inspect cdrx/pyinstaller-linux:python2'

                        sh 'pytest --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                    args '--privileged'
                }
            }
            steps {
                script {
                    try {
                        sh 'pyinstaller --onefile sources/add2vals.py'
                        echo 'SLEEP'
                        sleep(time:60, unit: "SECONDS")
                        input message: 'Finished using the website? (Click "Proceed" to continue)'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'dist/add2vals'
                }
            }
        }
    }
}