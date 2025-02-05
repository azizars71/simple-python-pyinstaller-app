pipeline {
    agent any

    environment {
        DOCKER_CONTAINER_NAME = 'my_python2_app'  // Change to your actual container name
        DEPLOY_DIR = '/app'  // Change to the directory where the app should be deployed inside the container
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                    args '--rm -v $PWD:/workspace -w /workspace'  // Mount workspace
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
                    args '--rm -v $PWD:/workspace -w /workspace'  // Mount workspace
                }
            }
            steps {
                script {
                    try {
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
                    args '--rm -v $PWD:/workspace -w /workspace'  // Mount workspace
                }
            }
            steps {
                script {
                    try {
                        sh 'pyinstaller --onefile sources/add2vals.py'
                        echo 'Waiting for deployment confirmation...'
                        sleep(time:60, unit: "SECONDS")
                        input message: 'Are you done yet? (Click "Proceed" to continue)'

                        // Copy the built artifact to the running Docker container
                        sh "docker cp dist/add2vals ${DOCKER_CONTAINER_NAME}:${DEPLOY_DIR}/add2vals"

                        // Restart the service inside the container if needed
                        sh "docker exec ${DOCKER_CONTAINER_NAME} chmod +x ${DEPLOY_DIR}/add2vals"
                        sh "docker exec ${DOCKER_CONTAINER_NAME} supervisorctl restart myapp"  // Adjust for your process manager

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
