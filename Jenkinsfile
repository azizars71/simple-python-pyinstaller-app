pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deploy') {
            agent any
            steps {
                script {
                    echo 'Deploying add2vals.py to localhost...'
                    
                    // Ensure deploy directory exists
                    sh 'mkdir -p /opt/myapp'

                    // Copy the script to a local directory
                    sh 'cp sources/add2vals.py /opt/myapp/'

                    // Give execution permissions
                    sh 'chmod +x /opt/myapp/add2vals.py'

                    // Run the script (optional, if required)
                    sh 'python /opt/myapp/add2vals.py'

                    echo 'Deployment to localhost completed successfully!'
                    
                    echo 'SLEEP'
                    sleep(time:60, unit: "SECONDS")
                    input message: 'Finished using the website? (Click "Proceed" to continue)'

                }
            }
        }
    }
}

