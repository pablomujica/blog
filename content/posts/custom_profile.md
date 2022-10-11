---
title: "PowerShell $Profile file - Adding Custom Functions to Your Default PowerShell Terminal"
date: 2022-10-10T21:44:31-03:00
draft: false
---

# Introduction

The idea of this article is to help with the management of custom functions for your daily tasks

I'll like to clarify that this is my personally preferred way to do this, it can be unnecessarily overcomplicated to some, but in the way that I tackle it, it makes the most sense for my workflow.

# Creation of the file

The first thing you have to do is create a base file, the $Profile file does not exists by default(an error by Microsoft IMO), so to create it you can do it with the command:
``` powershell
New-Item -Path $profile -ItemType "file" -Force
```

# Managing modules

PowerShell modules are .psm1 files where you define your functions, that way you can package all of the related functions in a single file, keeping neatly organized.

My preferred way to manage my modules is to add them to a .dotFiles folder in the root of my Path, that way I can centralize them in one place when needing to find them, and creating symbolic links to the places from where I maintain them, and backed up.

## Creation of the folder:
``` powershell
$dotFolder = New-Item -Path $env:HOMEPATH -Name .dotFiles -ItemType "directory"
```

## Creation of the SymbolicLink
With a SymbolicLink, that way the you only have one module file to modify, avoiding the need to synchronize of your personal folders with the ones your functions are in. This command needs Gsudo(explained [here]({{< ref "/posts/gsudo.md" >}})), an alternative is use a PowerShell console with administrative privileges. 

``` powershell
$psModule = "$env:HOMEPATH\MyFolder\dotFiles\CustomModules.psm1"
$syncLinkModule = $dotFolder.FullName+"\CustomModule.psm1"
gsudo New-Item -ItemType SymbolicLink `
               -Target $psModule `
               -Path $syncLinkModule
```

## Adding the module to the profile

For your modules be loaded to your sessions, you'll need to import them with the command [Import-Module](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/import-module?view=powershell-7.2).

Particularly a single-line way to add it would be:
``` powershell
Add-Content $profile ("Import-Module {0}" -f $syncLinkModule)
```

## Reloading your session

The functions will not be available right away, for that, you have to reload your profile file, which is pretty simple with the command:

``` powershell
.$profile
```

# Possible problem with this approach
A common problem with this approach is that your [Execution Policy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2), which controls the scripts you're allowed to run, its too restrictive. The way I have it set-up is for the LocalMachine scope, you can run file marked as RemoteSigned. If you are having problem to run a script file, you can unblock it using:

``` powershell
Unblock-File $Path
```

To change your execution policy, you have to use the [Set-ExecutionPolicy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-7.2) function, for example:

``` powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

Or in case you want to specify the scope of this execution policy:

``` powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

# Conclusion

So that's the way I manage my custom functions in PowerShell, it can seem complicated but in my opinion its the most modular and versatile way to do it.