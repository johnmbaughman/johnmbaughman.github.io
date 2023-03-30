---
layout: toc-page
title: Other Utilities
description: Utilities included with the script
dropdown: Articles
priority: 1
---
## Introduction

These utility functions are used in the main functions described above and are exposed as useful utilities outside of them.

### `Test-UserCredentials`

Does exactly that. It let's you know that the account is valid as well as the password.

* Parameters:
  * `-userCredential` - A `PSCredential` object containing a user name and password.

* Usage:

``` powershell
$secretString = Read-Host -Prompt "Enter your top secret password" -AsSecureString
$psCred = New-Object PSCredential("domain\username", $secureString)
Test-UserCredentials -userCredential $psCred
```

A `"domain\username"` example: `global\svc_Atlanta`

### `Install-Iis`

Does exactly that too. It runs through and installs the components most commonly used by our applications. Does not include the .Net Core hosting components (yet).

* Parameters:
  * None

* Usage:

``` powershell
Install-Iis
```

### `Test-PowerShellVersion`

Tests the PowerShell version used in the current session is of the correct versioning.

* Parameters:
  * `-MinimumVersion` - Must be greater than or equal to this version.
  * `-MaximumVersion` - Must be less than or equal to this version.

> The parameters cannot be combined.

* Usage:

``` powershell
Test-PowerShellVersion -MinimumVersion '7.3'
Test-PowerShellVersion -MaximumVersion '5.4'
```

### `Get-OSProductType`

Did you ever wonder what the product type of your Windows is? This will tell you.

* Parameters:
  * None

* Usage:

``` powershell
Get-OSProductType
```

### `Test-LocalAdministrator`

Tests if the provided account name is in the `Local\Administrators` group. This is not an Active Directory check, only on the current local machine.

* Parameters:
  * `-userAccount` - account name.  

> The account name *can* be an Active Directory domain account. Proper name identification is mandatory; there is no regex or wildcard usage provided.

* Usage:

``` powershell
Test-LocalAdministrator -userAccount "global\svc_Atlanta"
Test-LocalAdministrator -userAccount "DefaultAccount"
```

### `Get-WebSiteParameters`

Not really a utility per se, but still useful. If you want to see the base configuration of the components installed in [Install-ShopFloorWeb](./Installing-a-Shop-Floor-Web-Site.md#install-shopfloorweb), you can use this function.

* Parameters:
  * None

* Usage:

``` powershell
Get-WebSiteParameters | ConvertTo-Json
```

Converting this to JSON first will make it easier to view the data.