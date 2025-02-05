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
            steps {
                script {
                    try {
                        // Run the Python 2 container temporarily to build the binary
                        sh '''
                        docker run --rm -v $(pwd):/workspace -w /workspace \
                        python:2-alpine sh -c "apk add --no-cache gcc musl-dev libffi-dev && pip install pyinstaller==3.6 && pyinstaller --onefile sources/add2vals.py"
                        '''


                        echo 'Waiting for deployment confirmation...'
                        sleep(time:60, unit: "SECONDS")
                        input message: 'Are you done yet? (Click "Proceed" to continue)'

                        // Copy the built artifact to the running Docker container
                        sh "docker cp dist/add2vals ${DOCKER_CONTAINER_NAME}:${DEPLOY_DIR}/add2vals"

                        // Make it executable & restart if needed
                        sh "docker exec ${DOCKER_CONTAINER_NAME} chmod +x ${DEPLOY_DIR}/add2vals"
                        sh "docker exec ${DOCKER_CONTAINER_NAME} supervisorctl restart myapp"  // Adjust if needed

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
