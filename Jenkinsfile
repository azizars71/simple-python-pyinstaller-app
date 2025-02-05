pipeline {
    agent any

    environment {
        // ssh ke ec2
        SSH_CREDENTIALS = 'EC2cicdproj'
        EC2_USER = 'ec2-user'
        EC2_HOST = 'ec2-3-0-103-129.ap-southeast-1.compute.amazonaws.com'
        EC2_PATH = '/home/ec2-user/'
    }

    stages {
        // build with trycatch
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

        // stage with trycatch
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

        // deploy with trycatch
        // specially made for using python2 in 2025
        stage('Deploy') {
            steps {
                script {
                    try {
                        // temp python2 docker or IT WILL NOT EVEN RUN
                        sh '''
                        docker run --rm -v $(pwd):/workspace -w /workspace \
                        python:2-alpine sh -c "apk add --no-cache gcc musl-dev libffi-dev && pip install pyinstaller==3.6 && pyinstaller --onefile sources/add2vals.py"
                        '''
                        
                        // kriteria sleep
                        //echo 'Waiting for deployment confirmation...'
                        //sleep(time:60, unit: "SECONDS")

                        // kriteria input message
                        //input message: 'Are you done yet? (Click "Proceed" to continue)'

                        // copy to EC2
                        sshagent(credentials: [SSH_CREDENTIALS]) {
                            sh """
                                scp -o StrictHostKeyChecking=no dist/add2vals ${EC2_USER}@${EC2_HOST}:${EC2_PATH}
                                ssh ec2-user@ec2-3-0-103-129.ap-southeast-1.compute.amazonaws.com "logger 'add2vals berhasil di-copy dari Jenkins!'" | sudo tee -a /var/log/deploy.log"
                            """
                        }

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
