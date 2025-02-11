pipeline {
    agent {
        label 'windows'  // Use a Windows agent
    }

    environment {
        CHROME_VERSION = "133.0.0.0"  // Required Chrome version
        CHROMEDRIVER_VERSION = "133.0.0.0"  // Matching ChromeDriver version
        DOTNET_VERSION = "6.0.x"  // Set the required .NET Core version
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Set up .NET Core') {
            steps {
                script {
                    // Ensure .NET Core is available on the agent
                    withEnv(["DOTNET_ROOT=${tool 'dotnet'}"]) {
                        bat 'dotnet --version'
                    }
                }
            }
        }

        stage('Uninstall Existing Chrome') {
            steps {
                script {
                    bat '''
                    wmic product where "name like 'Google Chrome%%'" call uninstall /nointeractive || echo "No Chrome found"
                    '''
                }
            }
        }

        stage('Install Chrome 133') {
            steps {
                script {
                    bat """
                    echo Installing Chrome version ${CHROME_VERSION}...
                    curl -o chrome_installer.exe https://dl.google.com/release2/chrome/${CHROME_VERSION}/chrome_installer.exe
                    start /wait chrome_installer.exe /silent /install
                    """
                }
            }
        }

        stage('Download and Install ChromeDriver 133') {
            steps {
                script {
                    bat """
                    echo Installing ChromeDriver version ${CHROMEDRIVER_VERSION}...
                    curl -o chromedriver.zip https://storage.googleapis.com/chrome-for-testing-public/${CHROMEDRIVER_VERSION}/chromedriver-win64.zip
                    powershell -Command "Expand-Archive -Path chromedriver.zip -DestinationPath C:\\chromedriver"
                    set PATH=%PATH%;C:\\chromedriver
                    """
                }
            }
        }

        stage('Restore Dependencies') {
            steps {
                script {
                    bat 'dotnet restore'
                }
            }
        }

        stage('Build Project') {
            steps {
                script {
                    bat 'dotnet build --configuration Debug'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    bat 'dotnet test --logger trx'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/TestResults/*.trx', allowEmptyArchive: true
        }
        failure {
            echo "Build or tests failed! Check logs for details."
        }
    }
}
