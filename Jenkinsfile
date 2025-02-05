pipeline {
    agent any

    environment {
        // ssh ke ec2
        SSH_CREDENTIALS = 'Ubuntu-cicd-ec2'
        EC2_USER = 'ubuntu'
        EC2_HOST = 'ec2-47-129-32-198.ap-southeast-1.compute.amazonaws.com'
        EC2_PATH = '/home/ubuntu'
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

        stage('Manual Approval'){
            steps{
                script{
                    try{
                        // kriteria input message
                        input message: 'Lanjutkan ke tahap Deploy?'
                    }catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
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
                        
                        // ssh copy ke EC2
                        sshagent(credentials: [SSH_CREDENTIALS]) {
                            sh """
                                scp -o StrictHostKeyChecking=no dist/add2vals ${EC2_USER}@${EC2_HOST}:${EC2_PATH}
                                ssh ${EC2_USER}@${EC2_HOST} "sudo logger 'DEPLOY DARI JENKINS BERHASIL!' && sudo sh -c 'echo \"DEPLOY DARI JENKINS BERHASIL!\" >> /var/log/deploy-from-jenkins.log'"
                            """
                        }

                        // kriteria sleep
                        echo 'Sleep - 60 seconds'
                        sleep(time:60, unit: "SECONDS")

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
