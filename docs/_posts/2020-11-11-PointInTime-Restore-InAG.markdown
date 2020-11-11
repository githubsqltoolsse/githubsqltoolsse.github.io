---
layout: posts
title:  "Point in time restore in AG"
date:   2020-11-11 10:31:16 +0100
categories: powershell DBATools
tags: Powershell DBATools DisasterRecovery
---

Some introduction bla bla.

## Important notes

- Make sure log backups are turned of on all target servers.
- Verify that default path is correct on the db instances. 
   - They must be equal on all servers.
   - If not, the path must be provided in the restore command.
- Do restore as sa if you want sa to be owner. That makes it easier to get the owner correct from the beginning.
- On secondary do restore with no recovery.

## Steps included in restore

1. Turn of log backups on all servers
2. Start restore on primary server
3. Start restore on secondary server (This can be done while restore to primary is running)
4. Add database to availability group with join only.
5. Start log backups on all servers.

## Restore to primary


{% highlight powershell %}

##########################
#Do the restore. Primary
##########################

#Open message box to verify log backups turned off.
$msgBoxInput = [System.Windows.Forms.MessageBox]::Show("Have you turned off log backup on source and target servers?",'Verify log backups turned off','YesNo','Question')
switch  ($msgBoxInput) {
    'Yes' {
        Write-Host "Restore verified." -ForegroundColor Green
    }
    'No' {
        Write-host "Restore not verified breaking." -ForegroundColor Yellow
        RETURN
    }
}

#Open a dialog for adding the sa password.
$Credential = Get-Credential -Message "Give the credentials for sa account on new primary server" -UserName "sa"

$params = @{
    SqlInstance = "MyPrimaryServer\MyPrimaryInstance"
    Path = "\\MyBAckupPath\OldServer`$OldInstance\Database"
    MaintenanceSolutionBackup = $true
    DatabaseName = "Database" 
    NoRecovery = $false
    SqlCredential = $Credential
}
Restore-DbaDatabase @params

{% endhighlight %}


## Restore to secondary

{% highlight powershell %}

##########################
#Do the restore. Secondary
##########################

#Open message box to verify log backups turned off.
$msgBoxInput = [System.Windows.Forms.MessageBox]::Show("Have you turned off log backup on source and target servers?",'Verify log backups turned off','YesNo','Question')
switch  ($msgBoxInput) {
    'Yes' {
        Write-Host "Restore verified." -ForegroundColor Green
    }
    'No' {
        Write-host "Restore not verified breaking." -ForegroundColor Yellow
        RETURN
    }
}


$Credential = Get-Credential -Message "Give the credentials for sa account on new secondary server" -UserName "sa"

$params = @{
    SqlInstance = "MySecondaryServer\MySecondaryInstance"
    Path = "\\MyBackupPath\OldServer`$OldInstance\Database"
    MaintenanceSolutionBackup = $true
    DatabaseName = "Database"
    NoRecovery = $true #Must be true on secondary instance
    SqlCredential = $Credential
}
Restore-DbaDatabase @params

{% endhighlight %}
