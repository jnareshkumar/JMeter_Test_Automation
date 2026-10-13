
pipeline {
    agent { label 'windows' }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        JMETER_HOME = 'C:\NARESH\Perf Test Training\apache-jmeter-5.6.3'
        TEST_PLAN = 'Petstore.jmx'
        CSV_FILE = 'Petstore_InputData.CSV'
    }

    stages {
        stage('Checkout GitHub') {
            steps {
                checkout scm
                bat 'git log -1 --oneline'
            }
        }

        stage('Validate Files') {
            steps {
                bat '''
                    if not exist "%JMETER_HOME%\\bin\\jmeter.bat" (
                        echo ERROR: JMeter not found
                        exit /b 1
                    )

                    if not exist "%TEST_PLAN%" (
                        echo ERROR: JMX test plan not found
                        exit /b 1
                    )

                    if not exist "%CSV_FILE%" (
                        echo ERROR: CSV file not found
                        exit /b 1
                    )

                    "%JMETER_HOME%\\bin\\jmeter.bat" -v
                '''
            }
        }

        stage('Run JMeter Test') {
            steps {
                bat '''
                    if exist results rmdir /s /q results
                    mkdir results

                    call "%JMETER_HOME%\\bin\\jmeter.bat" ^
                      -n ^
                      -t "%TEST_PLAN%" ^
                      -l "results\\results.jtl" ^
                      -e ^
                      -o "results\\html-report"

                    if errorlevel 1 exit /b 1
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts(
                artifacts: 'results/**',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'JMeter test execution completed.'
        }

        failure {
            echo 'JMeter pipeline failed. Check Console Output.'
        }
    }
}
