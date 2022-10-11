---
title: "GSudo - Elevate your current PowerShell Session, and use it in the automation of your daily tasks"
date: 2022-10-10T21:51:43-03:00
draft: true
---

# https://github.com/gerardog/gsudo

Gsudo its a good alternative for the missing easy way to elevate the current console line in #Windows.

You can elevate scripts or commands that require admin permissions. An important cabiat its that it returns are just strings, so you need to convert the text to the appropriate types that you want to use.

# Install using Chocolatey

The most simple way of installing GSudo is via Chocolatey, a packet manager similar to Brew that enables a simple straightforward forward to install the software you need in windows, the same way that its been for years in other OS's.

``` shell
choco install gsudo
```

# Simple use

This is the basic example of his use, here you elevate the current shell, and pass the command to open the hosts file un your pc, that's a very common task that needs elevated permissions.

``` powershell
gsudo notepad $env:WINDIR\system32\drivers\etc\hosts
```

You can even go back on your errors in a way that helps you with previous commands that failed due to insufficient privileges:

``` shell
gsudo !!
```

As a good alternative, you can one-line it with:
``` powershell
gsudo "echo 'IP.IP.IP.IP          custom.domain' >> $env:WINDIR\system32\drivers\etc\hosts"
```

Why not even better? with a PowerShell function:
``` powershell
function Add-HostLine {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [String]
        $IP,
        [Parameter(Mandatory=$true)]
        [String]
        $Domain
)
    $isIP = [bool]( `
        $IP -as [ipaddress] -and ( `
            $IP.ToCharArray() | Where-Object {$_ -eq "."} `
	        ).count -eq 3 `
		)

    if ($isIP) {
        $argString = "echo {0}          {1}          # Added with `$posh and gsudo >> $env:WINDIR\system32\drivers\etc\hosts" -f $IP, $Domain
        gsudo -d $argString
        if ($LastExitCode -eq 999) {
            Throw "Not Elevated"
        } elseif ($LastExitCode) {
            Throw "Command Failed"
        } else {
            Write-Host ("Added the record {0} for {1}" -f $IP, $Domain)
        }
    } else {
        Throw "Not a valid ipv4 IP"
    }
}
```

This function can corroborate the supplied value is in the correct format for IPv4 IPs to be added to your host file, and add it to said file.

# Use it in every console

Now that you have your new function, you don't want to loose it, a simple way would be to add it to your $profile file, that is the file that is loaded every time that you open your console. 

The correct way for this is create a PowerShell Module file, where you define your functions, to load it with the Import-Module command, that way are available in the current console

``` powershell
    $CUSTOM_MODULE = 'C:\PATH\TO\YOUR\MODULE\FILE.psm1'
    Add-Content $profile ("`rImport-Module {0}" -f $CUSTOM_MODULE)
```

