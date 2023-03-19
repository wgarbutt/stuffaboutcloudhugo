---
title: Automated password resetting for ESXi Hosts
date: 2022-07-05
tags:
- VMWare
showtableofcontents: true
showtaxonomies: true
showsummary: true
summary: "Using a PowerCLI script to change 800+ ESXi hosts root passwords monthly" 

---

A few months ago I was tasked with finding a solution for resetting root passwords for 800+ VMWare ESXi hosts on a regular schedule.

We had recently completed a project to move our credential store to [Passwordstate][1] and had noticed it had some functionality around automating resetting and validating credentials.

I initially started looking at the built-in Linux scripts which utilises SSH connections, something we have disabled for our ESXi hosts for security.

Searching through forums I found a post where someone used PowerCLI to do the heavy lifting but found the post didnt quite give me everything I needed to complete the project.

I made some custom scripts that do a pretty good job at completing the solution.

Password reset and password validation scripts:

We need to talk about these custom scripts first, as we need the IDs of the script to fill in the JSON data for scripted host ingest

### Setting the ESXi Password Script:
```powershell
 Function Set-ESXiPassword {
   [CmdletBinding()]
   param (
      [String]$HostName,
      [String]$UserName,
      [String]$OldPassword,
      [String]$NewPassword
   )    
   try {
      $Connection = Connect-VIServer $HostName -User $UserName -Password $OldPassword
   } 
   catch {
      switch -wildcard ($error[0].Exception.ToString().ToLower()) {
         "*incorrect user*" { Write-Output "Incorrect username or password on host '$HostName'" break }
         "*" { write-output $error[0].Exception.ToString().ToLower() break }
      }
   }
   try {
      $change = Set-VMHostAccount -UserAccount $UserName -Password $NewPassword | out-string
      if ($change -like '*root*') {
         Write-Output "Success" 
      }
      else { 
         Write-Output "Failed" 
      }
      Disconnect-Viserver * -confirm:$false
   } 
   catch {
      switch -wildcard ($error[0].Exception.ToString().ToLower()) {
         "*not currently connected*" { Write-Output "It wasn't possible to connect to '$HostName'" break }
         "*weak password*" { Write-Output "Failed to execute script correctly against Host '$HostName' for the account '$UserName'. It appears the new password did not meet the password complexity requirements on the host." break }
         "*" { write-output $error[0].Exception.ToString().ToLower() break }
         default { Write-Output "Got here" }
      }
   }
}
   
Set-ESXiPassword -HostName '[HostName]' -UserName '[UserName]' -OldPassword '[OldPassword]' -NewPassword '[NewPassword]'   
```

This script logs into a host directly via powershell and powercli modules, Passwordstate passes the common variables such as OldPassword, NewPassword, Username etc. The Set-VMHostAccount command is actually doing all the heavy lifting here. The rest of the script is some simple error handling.

PowerCLI is baked into every ESXi host and only requires powershell to be open from the Passwordstate webserver to the host (port 443).

### Password Validation Script
```powershell
Function Validate-ESXiPassword {
    [CmdletBinding()]
    param (
        [String]$HostName,
        [String]$UserName,
        [String]$CurrentPassword
    )   
    $ErrorActionPreference = "Stop"
       
    try {   
        $Connection = Connect-VIServer $HostName -User $UserName -Password $CurrentPassword 
        if ($Connection.isconnected) {
            Write-Output "Success" 
        }
        else { 
            Write-Output "Failed" 
        }
    }   
     
    catch {
        switch -wildcard ($error[0].Exception.ToString().ToLower()) {
            "*incorrect user*" {
                Write-Output "Incorrect username or password on host '$HostName'" break
                Disconnect-VIServer $HostName -Force -Confirm:$false
            }
            default { Write-Output "Error is: $($error[0].Exception.message)" }                   
           
        }
    }
}
Validate-ESXiPassword -HostName '[HostName]' -UserName '[UserName]' -CurrentPassword '[CurrentPassword]'
```

This script simply attempts to connect to the host in question via Powershell.

The success criteria simply looks for the word root in the output, this may be foolish of me, but there isn't much of a result from the command to parse for a successful result

If the command fails it should be captured by my catch commands

### Host/Password Entry:

All of our hosts are domain joined so host discovery was rather straightforward enough by using the built in utility in Passwordstate. Unfortunately there was no easy way to automatically discover host accounts, but since we are only dealing with Root here we can script adding of password entries. You'll need to get your custom script IDs from the ones you created above. This is a one off script and took around one minute to add 800 hosts

```powershell
Connect-VIServer (your vcenter server)

$hostlist = get-vmhost 
$Creds = Get-Credential
$PasswordstateUrl = 'https://passwordstateurl/winapi/passwords'
 
  
  foreach ($hostname in $hostlist) {
   Write-Host "I am working on host $($Hostname.name)"
   $jsonData = '
    {
        "PasswordListID":"existingpasswordlistID",
        "Title":"' + $($hostname.name) + '",
        "UserName":"root",
        "password":"existingpassword",
        "hostname":"' + $($hostname.name) + '",
        "AccountTypeID": "34", (VMWare)
        "PasswordResetEnabled": false,
        "EnablePasswordResetSchedule": true,
        "ScriptID": "28",
        "HeartbeatEnabled": true,
        "ValidationScriptID": "22",
    }
    '
   Write-Host  $jsondata
    
   $result = Invoke-Restmethod -Method Post -Uri $PasswordstateUrl -ContentType "application/json" -Body $jsonData -Credential $Creds
   }
   Write-Host "Disconnecting vCenter"
   Disconnect-Viserver * -confirm:$false
```

 [1]: https://www.clickstudios.com.au