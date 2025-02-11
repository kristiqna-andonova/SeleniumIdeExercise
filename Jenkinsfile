pipeline {
    agent any

    environment {
        CHROME_VERSION = '133.0.6943.60'
        CHROMEDRIVER_VERSION = '133.0.6943.60'
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROMEDRIVER_PATH = 'D:\\chromedriver-win64.zip\\chromedriver-win64' 
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
                echo Checking if Google Chrome is installed...
                for /f "tokens=*" %%i in ('choco list --local-only ^| findstr /i "googlechrome"') do set FOUND=1
                if defined FOUND (
                    echo Uninstalling current Chrome...
                    choco uninstall googlechrome -y --skip-autouninstaller
                ) else (
                    echo Google Chrome is not installed. Skipping uninstall.
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
                bat '''
                echo Checking latest ChromeDriver version...
                powershell -command "$version = (Invoke-WebRequest -Uri 'https://googlechromelabs.github.io/chrome-for-testing/known-good-versions-with-downloads.json' -UseBasicParsing | ConvertFrom-Json).versions | Where-Object { $_.version -like '133*' } | Select-Object -First 1 -ExpandProperty version; echo Found ChromeDriver version: $version; $url = 'https://storage.googleapis.com/chrome-for-testing/' + $version + '/win64/chromedriver-win64.zip'; echo Downloading from $url; Invoke-WebRequest -Uri $url -OutFile chromedriver.zip -UseBasicParsing"
        
                echo Extracting ChromeDriver...
                powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath . -Force"
        
                echo Moving chromedriver.exe to %CHROME_INSTALL_PATH%
                move /Y chromedriver-win64\\chromedriver.exe "%CHROME_INSTALL_PATH%\\chromedriver.exe"
                '''
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
