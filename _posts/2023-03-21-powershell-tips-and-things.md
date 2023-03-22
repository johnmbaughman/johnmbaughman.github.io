---
title: PowerShell Tips And Things
description: Random PowerShell Tips. Just the tips.
categories: PowerShell
---
## TIP \#1043: Install A Module Package (.nupkg) In Offline Mode

Typically, we can install packages through the [online repository](http://powershellgallery.com), but due to recent circumstances and the tightening of security on our servers, this isn't feasible without a lot of headaches and red tape. To prevent this and also limit the number of packages arbitrarily installed on the servers, there is a manual process that will allow the installation of a needed package.

Introducing: _Install the NuGet package via a local repository_

1. **YOU MUST BE A LOCAL ADMINISTRATOR ON YOUR LOCAL MACHINE AS WELL AS AN ADMINISTRATOR ON THE SERVER!** All of these steps ***require*** these permissions.
2. On your local machine, run `Install-PackageProvider -Name NuGet -RequiredVersion 2.8.5.201 -Force` to install the provider from a computer with an internet connection.
3. After the installation, you can find the provider installed in `C:\Program Files\PackageManagement\ProviderAssemblies` – copy this folder to the server you are trying to install the package on. Make sure the path is the same as your local machine. You must copy it to your user folders first or use a UNC path to copy it to the `Program Files` folder. Either way, the folder structure on the server should match your local machine for this folder and its contents.
4. Start a new PowerShell session as an administrator (Right-click, Run as administrator) on the server.
5. Create a folder called `C:\PowerShellRepository`. This will identify the purpose of this folder for others.
6. Download and copy your `.nupkg` file(s) into `C:\PowerShellRepository`
7. In PowerShell run `Register-PSRepository -Name Local -SourceLocation C:\PowerShellRepository -InstallationPolicy Trusted`
8. You can list the packages available with `Find-Module -Repository Local`
9. Run `Install-Module -Name <YourModuleName> -Repository Local -Scope AllUsers` where `<YourModuleName>` is the name of your package as returned by the command in step 8.
10. Profit.

## TIP \#30002: Need the latest PowerShell?

You can find it here: [PowerShell for every system!](https://github.com/PowerShell/PowerShell/releases)

## TIP \#42: How to know you're an admin

`$null -ne (Get-LocalGroupMember -Group "Administrators" -Member <your fully qualified account name> -ErrorAction SilentlyContinue)`

Simple and no digging through UI dialogs. If your account is in there, then it will say `True`, otherwise, it will tell you something different. Usually, it's `False` but with AI becoming all the rage, it could be something else.

## TIP \#42.2: So you want to be an admin?

This tip won't help you (directly). Buuuut... You can add another account as a local admin. 

_**YOU**_ will need to be an admin (local or in a group that has local admin privileges) to do this, so if you're not don't bother. If you are, read on.

At an administrator PowerShell prompt (most of these tips require that if you couldn't tell), enter the following command, replacing `<theAccount>` with the account you are trying to add. If it's a domain account (ie.: a service account), make sure it is fully qualified (ex.: `domain\theAccount`), otherwise, if it's a local account, just make sure to enter it as it is in the Computer Management users list.

``` powershell
Add-LocalGroupMember -Group Administrators -Member <theAccount>
```

Sorry, it's so complicated. I didn't create PowerShell, I just enjoy it.

## TIP \#101: I need to hide a password on the command line. How do I do that?

Easy. Real easy. 

1. Create a `SecureString`. You can accomplish this feat in two ways. One still has the visible string on the command line, while the other prompts you for your password or other secure string (get it?). Let's go with that idea since that's the title of the tip you are reading about.

Go to a PowerShell prompt (you don't have to use an admin one now, but getting into the habit of always opening it will make your life easier) and type the following command to get a `SecureString` variable.

``` powershell
$secretString = Read-Host -Prompt "Enter your top secret password" -AsSecureString
```
Notice the `-AsSecureString` parameter, which will mask the password with asterisks (`*`) and return an encrypted string. 

>A note about that `SecureString` encryption: It uses the machine key to encrypt the string, so this means that it can only be used on the machine it is created. Don't try and pass it off to another machine in a remote session. It won't work.

Now, how do I use that `SecureString`? Well, one way is with a [`PSCredential`](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential?view=powershellsdk-7.3.0). A `PSCredential` can be passed into various other cmdlets and functions to allow automating logging into a system.

You create a `PSCredential` for a network or ActiveDirectory domain login using that `SecureString` we created earlier like this:

``` powershell
$psCred = New-Object PSCredential("domain\username", $secureString)
```

Now that you have a credential, you can use it for such things as website app pool setups, Windows service setups, and scheduled task setups, among other things.

This is an oversimplification of testing the credential, but here it is anyway. You'll need the ActiveDirectory feature (RSAT) installed to do this.

``` powershell
Import-Module ActiveDirectory
$DomainNetBIOS = $psCred.username.Split("\")[0]
$UserName = $psCred.username.Split("\")[1]
$Password = $psCred.GetNetworkCredential().password # still a SecureString 
$DomainFQDN = (Get-ADDomain $DomainNetBIOS).DNSRoot
$DomainObj = "LDAP://" + $DomainFQDN
$DomainBind = New-Object System.DirectoryServices.DirectoryEntry($DomainObj, $UserName, $Password)
$DomainName = $DomainBind.distinguishedName
Get-ADUser -Server $DomainFQDN -Properties LockedOut -Filter { sAMAccountName -eq $UserName }
```
A few lines to get there, but it's not too difficult and can be very rewarding when trying to make sure the account and password are correct before diving into a bigger process that can be cumbersome or impossible to clean up without a lot of work. But I digress...

And all in 1 simple step.

## TIP \#420: My boss said I need to send him an EXE file in his email. How do I get around the spambot?

First, I'm going to ignore any malicious intent you have and just go with you pretending to be funny.

Second, to do this without malicious intent you simply use your C# tools available to you in PowerShell to just read all the bytes of the EXE, convert them to Base64, and return a string. 

``` powershell
$exeBytes = [System.IO.File]::ReadAllBytes("C:\Windows\Notepad.exe")
$base64Exe = [System.Convert]::ToBase64String($exeBytes)
``` 

Now you can convert it back to a file by doing the opposite:

``` powershell
$base64Bytes = [System.Convert]::FromBase64String($base64Exe)
[System.IO.File]::WriteAllBytes("C:\temp\notepad.exe", $bytes)
``` 

Now, go forth and spam... I mean use this knowledge with great responsibility.

## TIP \#68: JSON: It's not just for C# anymore

Ok, since PowerShell is basically a wrapper around .Net (it's written in C#, after all), it is just for C#. But not really.

Did you know that you can use a JSON structure in PowerShell a whole lot easier than in C#? Technically, it only takes one line, but I'll break it down so it's easier to understand.

Using this JSON data, we'll read it into a `PSCustomObject` and show how to access the data in that object.

``` json
[
  {
    "carname": "Mustang",
    "manufacturer": "Ford"
  },
  {
    "carname": "Camaro",
    "manufacturer": "Chevrolet"
  },
  {
    "carname": "Challenger",
    "manufacturer": "Dodge"
  }
]
```

``` powershell
$jsonString = Get-Content -Path "c:\temp\cars.json"
$cars = $jsonString | ConvertFrom-Json -depth 4 
# The -depth parameter just insures that all levels are read. The default is 1024 in PowerShell 6+, but sometimes it doesn't quite work without it.
# If you're in PowerShell <6, you're SOL and things really don't behave well past 4-5 levels. You've been warned. Use the latest PowerShell.
Write-Host "$($cars[0].carname) is made by $($cars[0].manufacturer)"
Write-Host "$($cars[1].carname) is made by $($cars[1].manufacturer)"
Write-Host "$($cars[2].carname) is made by $($cars[2].manufacturer)"
```
Now, writing an object to a JSON file is just the other way around. 

``` powershell
$cars[1].carname = 'Corvette'
$jsonCars = $cars | ConvertTo-Json
$jsonCars | Set-Content -Path "c:\temp\changedcars.json"
```

Bam... It's all done without [Json.NET](https://www.newtonsoft.com/json).