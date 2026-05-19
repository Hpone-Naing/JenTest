name: JenTest with Web Deploy (Zero Downtime)

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build-and-deploy:
    runs-on: self-hosted
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Restore
        run: dotnet restore

      - name: Build
        run: dotnet build -c Release --no-restore

      - name: Test
        run: dotnet test --verbosity normal

      - name: Create IIS Site and App Pool if not exists
        shell: pwsh
        run: |
          $siteName = "jentest"
          $appPoolName = "DefaultAppPool"
          $sitePath = "D:\jentest"  # Change this to your desired website path
          $sitePort = 8080  # Change this to your desired port
          
          # Create website directory if it doesn't exist
          if (-not (Test-Path $sitePath)) {
            New-Item -ItemType Directory -Path $sitePath -Force
            Write-Host "✅ Created website directory: $sitePath"
          }
          
          # Check if App Pool exists
          $appPoolExists = & "$env:SystemRoot\System32\inetsrv\appcmd.exe" list apppool /name:"$appPoolName" 2>$null
          if (-not $appPoolExists) {
            Write-Host "Creating App Pool: $appPoolName"
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" add apppool /name:"$appPoolName" /managedRuntimeVersion:"v4.0"
            Write-Host "✅ App Pool created: $appPoolName"
          } else {
            Write-Host "✅ App Pool already exists: $appPoolName"
          }
          
          # Check if Site exists
          $siteExists = & "$env:SystemRoot\System32\inetsrv\appcmd.exe" list site /name:"$siteName" 2>$null
          if (-not $siteExists) {
            Write-Host "Creating IIS Site: $siteName"
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" add site /name:"$siteName" /physicalPath:"$sitePath" /bindings:"http/*:$sitePort:"
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" set site /site.name:"$siteName" /[path='/'].applicationPool:"$appPoolName"
            Write-Host "✅ IIS Site created: $siteName on port $sitePort"
          } else {
            Write-Host "✅ IIS Site already exists: $siteName"
          }

      - name: Stop IIS Site
        shell: pwsh
        run: |
          $siteExists = & "$env:SystemRoot\System32\inetsrv\appcmd.exe" list site /name:"jentest" 2>$null
          if ($siteExists) {
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" stop site /site.name:"jentest"
            Write-Host "✅ Site stopped"
          } else {
            Write-Host "⚠️ Site not found, skipping stop"
          }

      - name: Stop IIS App Pool
        shell: pwsh
        run: |
          $appPoolExists = & "$env:SystemRoot\System32\inetsrv\appcmd.exe" list apppool /name:"DefaultAppPool" 2>$null
          if ($appPoolExists) {
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" stop apppool /apppool.name:"DefaultAppPool"
            Write-Host "✅ App Pool stopped"
          } else {
            Write-Host "⚠️ App Pool not found, skipping stop"
          }

      - name: Publish to temp folder
        run: dotnet publish -c Release -o "D:\jentest" --no-build

      - name: Deploy to IIS using Web Deploy (AppOffline)
        shell: pwsh
        run: |
          $sourcePath = "D:\jentest"  # This should match the site path above
          $destinationPath = "D:\jentest"  # This should match the site path above
          
          # Copy files to website directory
          Write-Host "Copying files from $sourcePath to $destinationPath..."
          if (-not (Test-Path $destinationPath)) {
            New-Item -ItemType Directory -Path $destinationPath -Force
          }
          
          # Use robocopy for reliable file copying
          & robocopy $sourcePath $destinationPath /MIR /R:3 /W:10 /NP
          
          if ($LASTEXITCODE -le 7) {
            Write-Host "✅ Deployment completed successfully"
          } else {
            Write-Host "❌ Deployment failed with exit code: $LASTEXITCODE"
            exit 1
          }

      - name: Start IIS App Pool
        shell: pwsh
        run: |
          $appPoolExists = & "$env:SystemRoot\System32\inetsrv\appcmd.exe" list apppool /name:"DefaultAppPool" 2>$null
          if ($appPoolExists) {
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" start apppool /apppool.name:"DefaultAppPool"
            Write-Host "✅ App Pool started"
          }

      - name: Start IIS Site
        shell: pwsh
        run: |
          $siteExists = & "$env:SystemRoot\System32\inetsrv\appcmd.exe" list site /name:"jentest" 2>$null
          if ($siteExists) {
            & "$env:SystemRoot\System32\inetsrv\appcmd.exe" start site /site.name:"jentest"
            Write-Host "✅ Site started"
          } else {
            Write-Host "⚠️ Site not found, cannot start"
            exit 1
          }