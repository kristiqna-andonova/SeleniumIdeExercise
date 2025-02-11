pipeline {
    agent any

    environment {
        // Reference credentials stored in Jenkins
        GIT_CREDENTIALS = credentials('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') // Replace with the actual credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout code using Git and the credentials
                    git url: 'https://github.com/kristiqna-andonova/SeleniumIdeExercise.git', credentialsId: GIT_CREDENTIALS
                }
            }
        }

        stage('Set up .NET Core') {
            steps {
                // Set up .NET Core on Windows
                bat 'dotnet --version'
            }
        }

        stage('Uninstall current Chrome') {
            steps {
                // Uninstall the current version of Chrome on Windows
                bat 'echo Uninstalling current Chrome...'
                bat 'wmic product where "name like \'%Google Chrome%\'" call uninstall /nointeractive'
            }
        }

        stage('Install specific version of Chrome') {
            steps {
                // Install a specific version of Chrome (replace with actual version)
                bat '''
                    echo Installing specific version of Chrome...
                    SET CHROME_VERSION=132.0.6834.196
                    SET DOWNLOAD_URL=https://www.googleapis.com/download/storage/v1/b/chromium-browser-snapshots/o/Win%2F%2F$CHROME_VERSION%2Fchrome_installer.exe?generation=1627223094395930&alt=media
                    SET INSTALLER=chrome_installer.exe
                    curl -L %DOWNLOAD_URL% -o %INSTALLER%
                    start /wait %INSTALLER%
                    del %INSTALLER%
                '''
            }
        }

        stage('Download and Install ChromeDriver') {
            steps {
                // Download and install ChromeDriver (match version to installed Chrome)
                bat '''
                    echo Installing ChromeDriver...
                    SET CHROMEDRIVER_VERSION=132.0.0.0
                    SET CHROMEDRIVER_URL=https://chromedriver.storage.googleapis.com/%CHROMEDRIVER_VERSION%/chromedriver_win32.zip
                    SET CHROMEDRIVER_ZIP=chromedriver.zip
                    curl -L %CHROMEDRIVER_URL% -o %CHROMEDRIVER_ZIP%
                    tar -xf %CHROMEDRIVER_ZIP%
                    del %CHROMEDRIVER_ZIP%
                    SET PATH=%PATH%;%CD%\chromedriver
                '''
            }
        }

        stage('Restore Dependencies') {
            steps {
                // Restore .NET dependencies
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                // Build the .NET Core project
                bat 'dotnet build'
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests for the .NET Core project
                bat 'dotnet test'
            }
        }
    }
}
