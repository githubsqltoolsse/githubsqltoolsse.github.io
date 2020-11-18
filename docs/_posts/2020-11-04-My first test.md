---
layout: posts
title:  "Test with Jekyll!"
date:   2020-11-04 23:31:16 +0100
categories: testing
toc: true
toc_label: "My Table of Contents"
---

# First test

Hello there.

```powershell

If(-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent() ).IsInRole( [Security.Principal.WindowsBuiltInRole] 'Administrator')){ throw "Run command in Administrator PowerShell Prompt"}
If($PSVersionTable.PSVersion -lt (New-Object System.Version("3.0"))){ throw "The minimum version of Windows PowerShell that is required by the script (3.0) does not match the currently running version of Windows PowerShell." }

$CredsAzureDevopsServices = Get-Credential -Message "Apply your credentials to Devops Azure. Use your personal access token as password!"
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$NugetUrl = "https://pkgs.dev.azure.com/SwecoAB/ApplicationDevelopment/_packaging/SwecoDbaTools/nuget/v2"
$PackageSources = Get-PackageSource | Where-Object {$psitem.location -eq $NugetUrl}
if(-not $PackageSources)
{
    Register-PSRepository -Name "SwecoPowershellModules" -SourceLocation $NugetUrl -PublishLocation $NugetUrl -InstallationPolicy Trusted -Credential $credsAzureDevopsServices 
}

#Get last available module versions
$CurrentVersionSwDBATools = Find-Module SwecoDbaTools -Credential $credsAzureDevopsServices | Select-Object -ExpandProperty Version 

#Get Installed version SwecoDBATools
$InstalledVersionSwDBATools = Get-Module SwecoDbaTools -ListAvailable | Select-Object -ExpandProperty Version

#List versions
IF ($InstalledVersionSwDBATools -eq $CurrentVersionSwDBATools) {$FGColor = "Green"} else {$FGColor = "Red"}
Write-Host "Current SwecoDBATools version is: $InstalledVersionSwDBATools and available version is: $CurrentVersionSwDBATools" -ForegroundColor $FGColor

#Remove old module if newer exists
IF($null -ne $InstalledVersionSwDBATools -and $InstalledVersionSwDBATools -ne $CurrentVersionSwDBATools){
    write-host "Removing SwecoDBATools" -ForegroundColor Yellow
    Get-Process powershell* | Where-Object Id -NE $PID | Stop-Process   
    Uninstall-Module SwecoDBATools
}

#Install module if update available
IF($InstalledVersionSwDBATools -ne $CurrentVersionSwDBATools){
    write-host "Installing SwecoDBATools" -ForegroundColor Green
    Install-Module SwecoDbaTools -Repository SwecoPowershellModules -Credential $credsAzureDevopsServices
}

#Get current installed module versions
$InstalledVersionSwDBATools = Get-Module SwecoDbaTools -ListAvailable | Select-Object -ExpandProperty Version

#List versions
IF ($InstalledVersionSwDBATools -eq $CurrentVersionSwDBATools) {$FGColor = "Green"} else {$FGColor = "Red"}
Write-Host "Current SwecoDBATools version is: $InstalledVersionSwDBATools and available version is: $CurrentVersionSwDBATools" -ForegroundColor $FGColor

```