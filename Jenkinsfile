pipeline {
    agent any

    environment {
        CHROME_VERSION = '133.0.6943.60'
        CHROMEDRIVERS_VERSION = '133.0.6943.60'
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROMEDRIVER_PATH = '"D:\\chromedriver-win64.zip\\chromedriver-win64"'
        GIT_SSH_COMMAND = 'ssh -i "C:/Users/Dell/.ssh/id_ed25519"'
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                withCredentials([sshUserPrivateKey(credentialsId: 'github_ssh', keyFileVariable: 'SSH_KEY')]) {
                    bat 'git clone https://github.com/kristiqna-andonova/SeleniumIdeExercise.git'
                }
            }
        }

        stage('Set up .NET Core') {
            steps {
                bat '''
                dotnet --version | findstr "6.0" > nul
                if %ERRORLEVEL% NEQ 0 (
                    echo Installing .Net SDK 6.0...
                    choco install dotnet-sdk -y --version=6.0.100
                ) else (
                    echo .Net SDK 6.0 is already installed. Skipping installation.
                )
                '''
            }
        }

        stage('Uninstall current Chrome') {
            steps {
                bat '''
                echo Checking if Google Chrome is installed via Chocolatey...
                choco list --local-only | findstr /i "googlechrome" 1>nul
                if %ERRORLEVEL% NEQ 0 (
                    echo Google Chrome is not installed via Chocolatey, skipping uninstall.
                ) else (
                    echo Uninstalling current Chrome...
                    choco uninstall googlechrome -y --skip-autouninstaller
                )
                '''
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
