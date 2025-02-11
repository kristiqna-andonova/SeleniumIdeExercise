pipeline {
    agent any

    environment {
        // Reference credentials stored in Jenkins
        CHROME_VERSION = '133.0.6943.60'
        CHROMEDRIVERS_VERSION = '133.0.6943.60'
        CHROME_INSTALL_PATH = 'C:\Program Files\Google\Chrome\Application'
        CHROMEDRIVER_PATH = '"D:\chromedriver-win64.zip\chromedriver-win64"'// Replace with the actual credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout code using Git and the credentials
                    git credentialsId: 'github-credentials', batch: 'master', url: 'https://github.com/kristiqna-andonova/SeleniumIdeExercise.git'
                }
            }
        }

        stage('Set up .NET Core') {
            steps {
                // Set up .NET Core on Windows
                bat """
                echo Installing .Net SDK 6.0
                choco install dotnet-sdk -y --version=6.0.100
                """
            }
        }

        stage('Uninstall current Chrome') {
            steps {
                // Uninstall the current version of Chrome on Windows
                bat '''
                echo Uninstalling current Chrome...
                choco uninstall googlechrome -y
                '''
                
            }
        }

        stage('Install specific version of Chrome') {
            steps {
                // Install a specific version of Chrome (replace with actual version)
                bat '''
                    echo Installing specific version of Chrome...
                    choco install googlechore --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums
                '''
            }
        }

        stage('Download and Install ChromeDriver') {
            steps {
                // Download and install ChromeDriver (match version to installed Chrome)
                bat '''
                echo Downloading ChromeDriver version %CHROMEDRIVER_VERSION%
                powershell -command "Invoke-WebRequest -Uri https://chromedriver.storage.googleapis.com/%CHROMEDRIVER_VERSION%/chromedriver_win32.zip -OutFile chromedriver.zip -UseBasicParsing"
                powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath ."
                powershell -command "Move-Item -Path .\\chromedriver.exe -Destination '%CHROME_INSTALL_PATH%\\chromedriver.exe' -Force"
                '''
            }
        }

        stage('Restore Dependencies') {
            steps {
                // Restore .NET dependencies
                bat 'dotnet restore SeleniumIde.sln'
            }
        }

        stage('Build') {
            steps {
                // Build the .NET Core project
                bat 'dotnet build SeleniumIde.sln --configuration Release'
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests for the .NET Core project
                bat 'dotnet test SeleniumIde.sln --loger "trx;LogFileNmae=TestResults.trx"'
            }
        }
    }
}
