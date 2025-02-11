pipeline {
    agent any

    environment {
        CHROME_VERSION = '133.0.6943.60'
        CHROMEDRIVERS_VERSION = '133.0.6943.60'
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROMEDRIVER_PATH = '"D:\\chromedriver-win64.zip\\chromedriver-win64"'
    }

    stages {
        stage('SSH Agent') {
            steps {
                sshagent(['github_ssh']) { // Ensure the 'github_ssh' credential ID is correctly set in Jenkins
                    script {
                        // Git checkout using SSH
                        git credentialsId: 'github_ssh', branch: 'master', url: 'git@github.com:kristiqna-andonova/SeleniumIdeExercise.git'
                    }
                }
            }
        }

        stage('Set up .NET Core') {
            steps {
                bat 'echo Installing .Net SDK 6.0'
                bat 'choco install dotnet-sdk -y --version=6.0.100'
            }
        }

        stage('Uninstall current Chrome') {
            steps {
                bat 'echo Uninstalling current Chrome...'
                bat 'choco uninstall googlechrome -y'
            }
        }

        stage('Install specific version of Chrome') {
            steps {
                bat 'echo Installing specific version of Chrome...'
                bat 'choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums'
            }
        }

        stage('Download and Install ChromeDriver') {
            steps {
                bat 'echo Downloading ChromeDriver version %CHROMEDRIVER_VERSION%'
                bat 'powershell -command "Invoke-WebRequest -Uri https://chromedriver.storage.googleapis.com/%CHROMEDRIVER_VERSION%/chromedriver_win32.zip -OutFile chromedriver.zip -UseBasicParsing"'
                bat 'powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath . "'
                bat 'powershell -command "Move-Item -Path .\\chromedriver.exe -Destination \'%CHROME_INSTALL_PATH%\\chromedriver.exe\' -Force"'
            }
        }

        stage('Restore Dependencies') {
            steps {
                bat 'dotnet restore SeleniumIde.sln'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build SeleniumIde.sln --configuration Release'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'dotnet test SeleniumIde.sln --logger "trx;LogFileName=TestResults.trx"'
            }
        }
    }
}
