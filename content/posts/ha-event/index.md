---
title: HA Event List
date: 2023-03-21
tags:
- VMWare
- Powershell
showtableofcontents: true
showtaxonomies: true
showsummary: true
summary: "Using a PowerCLI script to retrieve a list of VMs affected by an HA event" 
---

As part of my day job managing a VMWare cloud platform with approximately 800 hosts, one of the challenges we face is responding quickly when an ESXi host fails. Inevitably, these kinds of events can occur, and when they do, it's critical that we inform our customers which of their virtual machines were affected as soon as possible.

In the past, identifying the impacted VMs could be a time-consuming process involving manually sifting through pages of event logs. However, to streamline this process and reduce the response time, I created a PowerShell script that automatically connects to our vCenter and identifies any VMs that were affected by a recent host failure.


## The script

```powershell
# Connect to vCenter Server
Connect-VIServer <vCenter Server>

# Get a list of VMs that were affected by a host failure and restarted on another host
$failedVMs = Get-VIEvent -Start (Get-Date).AddDays(-1) -MaxSamples 100000 -Type Warning | Where-Object {$_.FullFormattedMessage -match "restarted"} | Select-Object -ExpandProperty Vm

# Group the VMs by agency
$vmGroups = @{}
foreach ($vm in $failedVMs) {
    $agency = $vm.Name -replace '^(.{3})_.*', '$1'
    if ($vmGroups.ContainsKey($agency)) {
        $vmGroups[$agency] += @($vm.Name)
    } else {
        $vmGroups[$agency] = @($vm.Name)
    }
}

# Output the list of affected VMs grouped by agency
Write-Host "The following agencies were affected by a host failure and restarted on another host:"
foreach ($agency in $vmGroups.Keys) {
    Write-Host "${agency}:"
    $vmGroups[$agency] | ForEach-Object { $_ -replace '\s*\([^)]*\)' }
    Write-Host ""
}

# Disconnect from vCenter Server
Disconnect-VIServer -Confirm:$false

```

Lets break it down. 

`Connect-VIServer <vCenter Server>` - This line establishes a connection to the vCenter Server

`$failedVMs = Get-VIEvent -Start (Get-Date).AddDays(-1) -MaxSamples 100000 -Type Warning | Where-Object {$_.FullFormattedMessage -match "restarted"} | Select-Object -ExpandProperty Vm` - This line retrieves a list of virtual machines that were affected by a host failure and subsequently restarted on another host. It does this by using the Get-VIEvent cmdlet to retrieve events of type "Warning" that occurred within the past day, filtering for events whose `FullFormattedMessage` property includes the word "restarted", and selecting the affected virtual machines using the `Select-Object` cmdlet with the `-ExpandProperty` parameter.

`$vmGroups = @{}` - This line initialises an empty hash table, which will be used to group the affected virtual machines by agency.

`foreach ($vm in $failedVMs) {...}` - This loop iterates over each of the affected virtual machines in the `$failedVMs` array and performs the following steps:

`$agency = $vm.Name -replace '^(.{3})_.*', '$1'` - This line extracts the three-letter agency code from the name of the virtual machine using a regular expression. Specifically, it uses the `-replace` operator to replace everything in the virtual machine name after the first underscore character with an empty string.

`if ($vmGroups.ContainsKey($agency)) {...} else {...}` - The purpose of this conditional statement is to group the VMs by their associated agency. It checks whether the hash table $vmGroups already has an entry for the agency of the current virtual machine. If the agency already exists as a key in $vmGroups, the script adds the name of the current virtual machine to the array associated with that key. If the agency doesn't exist as a key, the script creates a new key for the agency and initializes its associated array with the name of the current virtual machine.

In simpler terms, the script is organizing the virtual machines by the agency they belong to. If the script has seen a virtual machine from a particular agency before, it adds the current virtual machine to that agency's list. If it hasn't seen a virtual machine from that agency before, it creates a new list for that agency and adds the current virtual machine to it.

`Write-Host "The following agencies were affected by a host failure and restarted on another host:"` - This line prints a header indicating the start of the list of affected agencies and virtual machines.

`foreach ($agency in $vmGroups.Keys) {...}` - This loop iterates over each of the keys in the $vmGroups hash table (i.e., the agency codes) and performs the following steps:

`Write-Host "${agency}:"` - This line prints the current agency code (with a colon) as a heading for the list of virtual machines associated with that agency.

`$vmGroups[$agency] | ForEach-Object { $_ -replace '\s*\([^)]*\)' }` - This line uses the `ForEach-Object` cmdlet to iterate over each of the virtual machine names associated with the current agency and remove any text in parentheses (e.g., the VM UUID). It does this using the `-replace` operator with a regular expression that matches any whitespace characters followed by a left parenthesis, followed by any number of characters that are not a right parenthesis, followed by a right parenthesis. It replaces any matches with an empty string.

`Write-Host ""` - This line prints a blank line between each agency to improve readability.

`Disconnect-VIServer -Confirm:$false` - This line terminates the connection to the vCenter Server using the Disconnect-VIServer cmdlet.


## Conclusion 
Now, when an ESXi host fails, I can simply run the script, which quickly returns a list of the impacted VMs grouped by the agency to which they belong. This has proven to be an effective way to communicate critical information to our customers in a timely manner, and has helped to streamline our incident response process overall.

By using this script, I've been able to reduce the time required to identify impacted VMs and inform our customers, while also minimizing the risk of human error.