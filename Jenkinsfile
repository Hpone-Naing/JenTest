\pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }
        
        stage('Build') {
            steps {
                bat 'dotnet build -c Release --no-restore'
            }
        }
        
        stage('Test') {
            steps {
                bat 'dotnet test --verbosity normal'
            }
        }
        
        stage('Deploy to IIS') {
            steps {
                powershell '''
                    Import-Module WebAdministration -ErrorAction SilentlyContinue
                    
                    $siteName = "jentest"
                    $appPoolName = "DefaultAppPool"
                    $sitePath = "D:\\jentest"
                    $sitePort = 9090
                    
                    # Create directory if needed
                    if (-not (Test-Path $sitePath)) {
                        New-Item -ItemType Directory -Path $sitePath -Force
                    }
                    
                    # Create App Pool if needed
                    if (-not (Get-ChildItem IIS:\\AppPools | Where-Object {$_.Name -eq $appPoolName})) {
                        New-WebAppPool -Name $appPoolName
                        Write-Host "Created App Pool: $appPoolName"
                    }
                    
                    # Create Site if needed
                    if (-not (Get-Website -Name $siteName)) {
                        New-Website -Name $siteName -Port $sitePort -PhysicalPath $sitePath -ApplicationPool $appPoolName
                        Write-Host "Created Website: $siteName on port $sitePort"
                    } else {
                        Write-Host "Website already exists: $siteName"
                    }
                    
                    # Stop and start to ensure clean deployment
                    Stop-Website -Name $siteName -ErrorAction SilentlyContinue
                    Stop-WebAppPool -Name $appPoolName -ErrorAction SilentlyContinue
                    
                    # Publish
                    dotnet publish -c Release -o "$sitePath" --no-build
                    
                    # Start everything
                    Start-WebAppPool -Name $appPoolName
                    Start-Website -Name $siteName
                    
                    Write-Host "Deployment complete! Site available at http://localhost:$sitePort"
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}