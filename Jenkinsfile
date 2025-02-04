node {
    def workspace = pwd()

    //bagian build stage
    stage('Build') {
        //eksekusi build stage
        docker.image('python:2-alpine').inside {
            sh 'mkdir -p sources'
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    //bagian test stage
    stage('Test') {
        sh 'pip install pytest'
        //eksekusi test stage
        docker.image('qnib/pytest').inside {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            } finally {
                junit 'test-reports/results.xml'
            }
        }
    }

    //DEPLOY
    stage('Deploy') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile sources/add2vals.py'
        }

        //artefak
        postBuildSuccess {
            archiveArtifacts 'dist/add2vals'
        }
    }
}
