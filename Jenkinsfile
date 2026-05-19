pipeline {
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
        
        stage('Create IIS Site and App Pool if not exists') {
            steps {
                powershell '''
                    $siteName = "jentest"
                    $appPoolName = "DefaultAppPool"
                    $sitePath = "D:\\jentest"
                    $sitePort = 8080
                    
                    # Create website directory if it doesn't exist
                    if (-not (Test-Path $sitePath)) {
                        New-Item -ItemType Directory -Path $sitePath -Force
                        Write-Host "Created website directory: $sitePath"
                    }
                    
                    # Check if App Pool exists
                    $appPoolExists = & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" list apppool /name:"$appPoolName" 2>$null
                    if (-not $appPoolExists) {
                        Write-Host "Creating App Pool: $appPoolName"
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" add apppool /name:"$appPoolName" /managedRuntimeVersion:"v4.0"
                        Write-Host "App Pool created: $appPoolName"
                    } else {
                        Write-Host "App Pool already exists: $appPoolName"
                    }
                    
                    # Check if Site exists
                    $siteExists = & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" list site /name:"$siteName" 2>$null
                    if (-not $siteExists) {
                        Write-Host "Creating IIS Site: $siteName"
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" add site /name:"$siteName" /physicalPath:"$sitePath" /bindings:"http://*:${sitePort}:"
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" set site /site.name:"$siteName" /[path='/'].applicationPool:"$appPoolName"
                        Write-Host "IIS Site created: $siteName on port $sitePort"
                    } else {
                        Write-Host "IIS Site already exists: $siteName"
                    }
                '''
            }
        }
        
        stage('Stop IIS Site') {
            steps {
                powershell '''
                    $siteExists = & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" list site /name:"jentest" 2>$null
                    if ($siteExists) {
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" stop site /site.name:"jentest"
                        Write-Host "Site stopped"
                    } else {
                        Write-Host "Site not found, skipping stop"
                    }
                '''
            }
        }
        
        stage('Stop IIS App Pool') {
            steps {
                powershell '''
                    $appPoolExists = & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" list apppool /name:"DefaultAppPool" 2>$null
                    if ($appPoolExists) {
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" stop apppool /apppool.name:"DefaultAppPool"
                        Write-Host "App Pool stopped"
                    } else {
                        Write-Host "App Pool not found, skipping stop"
                    }
                '''
            }
        }
        
        stage('Publish to folder') {
            steps {
                bat 'dotnet publish -c Release -o "D:\\jentest" --no-build'
            }
        }
        
        stage('Start IIS App Pool') {
            steps {
                powershell '''
                    $appPoolExists = & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" list apppool /name:"DefaultAppPool" 2>$null
                    if ($appPoolExists) {
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" start apppool /apppool.name:"DefaultAppPool"
                        Write-Host "App Pool started"
                    }
                '''
            }
        }
        
        stage('Start IIS Site') {
            steps {
                powershell '''
                    $siteExists = & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" list site /name:"jentest" 2>$null
                    if ($siteExists) {
                        & "$env:SystemRoot\\System32\\inetsrv\\appcmd.exe" start site /site.name:"jentest"
                        Write-Host "Site started"
                    } else {
                        Write-Host "Site not found, cannot start"
                        exit 1
                    }
                '''
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}