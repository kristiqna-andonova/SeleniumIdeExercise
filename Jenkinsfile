pipeline {
    agent any

    environment {
        // Define the Chrome and Chromedriver versions
        CHROME_VERSION = '133.0.6943.60'
        CHROMEDRIVER_VERSION = '133.0.6943.60'

        // Define installation paths
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROMEDRIVER_PATH = 'D:\\chromedriver-win64'

        // Set the Git SSH command to use a specific SSH key
        GIT_SSH_COMMAND = 'ssh -i "C:/Users/Dell/.ssh/id_ed25519"'
    }

    stages {
        // Checkout the repository using SSH credentials
        stage('Checkout') {
            steps {
                deleteDir()
                withCredentials([sshUserPrivateKey(credentialsId: 'github_ssh', keyFileVariable: 'SSH_KEY')]) {
                    bat 'git clone https://github.com/kristiqna-andonova/SeleniumIdeExercise.git'
                }
            }
        }

        // Install .NET SDK 6.0
        stage('Set up .NET Core') {
            steps {
                bat 'echo Installing .Net SDK 6.0...'
                bat 'choco install dotnet-sdk --version=6.0.100 -y'
            }
        }

        // Uninstall the current version of Chrome if installed
        stage('Uninstall current Chrome') {
            steps {
                bat 'echo Checking if Google Chrome is installed via Chocolatey...'
                bat '''
                choco list --local-only | findstr /i "googlechrome" > nul
                if %errorlevel% neq 0 (
                    echo Google Chrome is not installed via Chocolatey, skipping uninstall.
                ) else (
                    echo Uninstalling current Chrome...
                    choco uninstall googlechrome -y --skip-autouninstaller
                )
                '''
            }
        }

        // Install the specified version of Chrome
        stage('Install specific version of Chrome') {
            steps {
                bat 'echo Installing Chrome version %CHROME_VERSION%...'
                bat 'choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums'
            }
        }

        // Download and install the specific version of ChromeDriver
        stage('Download and Install ChromeDriver') {
            steps {
                bat 'echo Downloading ChromeDriver version %CHROMEDRIVER_VERSION%...'
                bat 'powershell -command "Invoke-WebRequest -Uri https://chromedriver.storage.googleapis.com/%CHROMEDRIVER_VERSION%/chromedriver-win64.zip -OutFile chromedriver.zip -UseBasicParsing"'
                bat 'powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath %CHROMEDRIVER_PATH% -Force"'
                bat 'powershell -command "Move-Item -Path %CHROMEDRIVER_PATH%\\chromedriver.exe -Destination \'%CHROME_INSTALL_PATH%\\chromedriver.exe\' -Force"'
            }
        }

        // Restore dependencies using .NET
        stage('Restore Dependencies') {
            steps {
                bat 'dotnet restore SeleniumIde.sln'
            }
        }

        // Build the solution in release mode
        stage('Build') {
            steps {
                bat 'dotnet build SeleniumIde.sln --configuration Release'
            }
        }

        // Run tests with the .NET test command and save the results
        stage('Run Tests') {
            steps {
                bat 'dotnet test SeleniumIde.sln --logger "trx;LogFileName=TestResults.trx"'
            }
        }
    }
}
