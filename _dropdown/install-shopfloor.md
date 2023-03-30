---
layout: toc-page
title: Install-ShopFloorWeb
description: The grit about running the script to intall a website
dropdown: Articles
priority: 2
---

## Introduction

Installing a shop floor website is pretty straightforward. All you'll need is the module file and administrator access on the server.

The base website components to be installed are defined in a configuration that is part of the module. There should be no need to modify this information, however, it can be retrieved with [`Get-WebSiteParameters`](./Other-Utilities.html#get-websiteparameters).

[[_TOC_]]

## Loading The Module

If this is not the first time running the module on this server, skip number 1. If you know what you are doing then skip on down to [PowerShell 7.3+ Installation Check](#powershell-7.3%2B-installation-check). If none of that applies to your situation, continue on here.

1. [Download](/.attachments/ShopFloorServerUtilities-dba2ee1e-8337-4ab1-8440-539387ae3ca9.zip), unzip and save the module file somewhere you can find it. The `Downloads` folder is usually not a good place.
2. Open PowerShell prompt as Administrator. This can easily be done on Windows Server 2016 and above by right-clicking on the start button and selecting `Windows PowerShell (Admin)`.
> Hint: Get used to the idea of running PowerShell as an admin. Most of what PowerShell is really used for is admin functions that can only be performed in an admin session. All functions documented here require an admin session.

> If you don't see `Windows PowerShell (Admin)`:
>   * Right-click on the taskbar and select `Taskbar Settings`
>   * Turn on `Replace Command Prompt with Windows PowerShell in the menu when I right-click the start button or press Windows key+X`
3. Change directories to the folder where you saved the module: ex: `cd \mymodulesavefolder`

``` powershell
Import-Module .\ShopFloorServerUtilities.psm1 -force
```

4. Keep going...

### PowerShell 7.3+ Installation check

To run the website installation function, you need to use PowerShell 7.3 or later. If you know that PowerShell 7.3 or later is installed, you can skip to [PowerShell 7.3+ Administrator Console](#powershell-7.3%2B-administrator-console). 

> You can [test the current PowerShell session](./Other-Utilities.md#%60test-powershellversion%60) for the required version.

> This step can also be used to check for any updates that are available for a PowerShell installation.

1. At the PowerShell prompt you opened previously, enter `Install-PowerShell`. You should see similar below.

``` powershell
Install-PowerShell
WARNING: MSG:UnableToDownload «https://go.microsoft.com/fwlink/?LinkID=627338&clcid=0x409» «»
WARNING: Unable to download the list of available providers. Check your internet connection.
Get-Package : No package found for 'PowerShell*'.
At C:\Users\jbaughma\Desktop\ShopFloorServerUtilities.psm1:17 char:18
+ ... currentPS = Get-Package -ProviderName MSI -Name 'PowerShell*' | Selec ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Microsoft.Power...lets.GetPackage:GetPackage) [Get-Package], Exception
    + FullyQualifiedErrorId : NoMatchFound,Microsoft.PowerShell.PackageManagement.Cmdlets.GetPackage

Current PowerShell (Core) installed: **Not Installed**
Installing PowerShell-7.3.3-win-x64.msi in the background. Please wait before proceeding.
```

2. You'll be prompted to install PowerShell via the installer dialog. 
3. Once the installer completes, you can close the current PowerShell window.

## Install-ShopFloorWeb

### PowerShell 7.3+ Administrator Console

Now, here's the cool thing: 

* Getting to PowerShell in a particular folder is greatly simplified
* You have a new command history that remembers things from the last session
* There's "historic" predictive text which means you can start typing and PowerShell will display the last command that matches what you executed (just hit tab and it will complete it and then edit it). 

To go do the installation:

1. In File Explorer, go to the folder where the module is located.
2. Right-click on the folder area, not a file, and select `PowerShell 7` > `Open here as Administrator`
3. Tap the up arrow key until you see the `Import-Module` command you entered earlier. If you don't see it, just go ahead and enter it again.

You are now ready to install a website.

Enter `Install-ShopFloorWeb` and these parameters:
   * `-siteName [plant name]` or `-site [plant name]`
      * If this is a QA site, **do not** add QA to the site name. See `-isQA` below.
      * Example site names:
         * `MtPleasantEast`
         * `Worthington`
         * `ChattanoogaDebone`
   * `-accountName [domain\serviceAccount]` or `-account [domain\serviceAccount]`
      * The account name the site will run as **must** be an Active Directory account. 
      * The format to enter the account name is `[domain name]\[service account name]`
      * Example accounts:
         * `global\svc_sfmtpleasanteast`
         * `global\svc_sfworthington`
   * `-sslCity [plant city]`
      * This is the city where the plant is located.
      * If QA, the city is generally `Richardson`
      * If the city is two words like `Mount Pleasant`, wrap the city in single or double quotes or combine the name parts into a single word.
   * `-sslStateAbbrv [state abbreviation]`
      * This is the two-letter abbreviation for the state. 
      * If QA, the state abbreviation is generally `TX` as that is where `Richardson` (or `Carrollton`) is located.
   * [Optional] `-sslCountry`
      * This is the country abbreviation.
      * The default value is "US".
   * [Optional] `-isQa`
      * This switch designates this installation as a QA server installation and will handle the naming and folder locations automatically. This is why you do not add the 'QA' to `-siteName`, we got you covered.
         * Site name will have `QA` added.
         * Installation folders will use the plant name where needed.

Things should be looking similar to the below example for a QA installation.

``` powershell
Import-Module .\ShopFloorServerUtilities.psm1 -force
Install-ShopFloorWeb -siteName Aibonito -account global\svc_sfaibonito -isQa -sslCity Richardson -sslStateAbbrv TX
```

And for PRD. (Puerto Rico is considered within the US.)

``` powershell
Import-Module .\ShopFloorServerUtilities.psm1 -force
Install-ShopFloorWeb -siteName Aibonito -account global\svc_sfaibonito -sslCity Aibonito -sslStateAbbrv PR
```

And for a plant in Canada, for example.

``` powershell
Import-Module .\ShopFloorServerUtilities.psm1 -force
Install-ShopFloorWeb -siteName Brooks -account global\svc_sfbrooks -sslCity Brooks -sslStateAbbrv AB -sslCountry CA
```

Next, we'll discuss the steps taken to automate the installation and the steps you'll need to perform next. Don't worry, there isn't a lot left to actually do despite how much text is in this document.

### IIS Installation check

The first thing the script does is check for the proper IIS components to be installed. This step will *always* be performed. There is an additional module installed that cannot be downloaded from the PowerShell repository on the internet. This step will establish an internal repository on the network and install the `IisAdministration` module. 

> This step is also available as an optional "[other utility](./Other-Utilities.md#%60install-iis%60)" for those times when you don't need to install a website.

> A word about PowerShell 7, Windows Server 2016+ compatibility, and this message:

 ``` text
WARNING: Module ServerManager is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
```

> This is an "in progress" type message that shows Microsoft still has a lot to resolve with PowerShell, .Net Core, and Windows. Since PowerShell 5 runs on .Net Framework 4 and most of the server management modules, including IIS, are also dependent on that version, PowerShell runs things in a compatibility mode for version 7 and later.  
> Since it is in compatibility mode, and PowerShell is in constant development, this message is kind of a bug in the compatibility function. If you were to add the recommended `-SkipEditionCheck` to the `Import-Module` call in the script, nothing would work at all.  
> This message is included in the examples for clarity of what you should currently see as of this writing. This can be seen in the various locations where it will occur and is nothing to worry about when running this module.

``` text
WARNING: Module ServerManager is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Checking/Installing Web-Server...
Web-Server previously installed. Skipping...
Checking/Installing Web-Http-Redirect...
Web-Http-Redirect previously installed. Skipping...
Checking/Installing Web-Windows-Auth...
Web-Windows-Auth previously installed. Skipping...
Checking/Installing Web-Net-Ext45...
Web-Net-Ext45 previously installed. Skipping...
Checking/Installing Web-Asp-Net45...
Web-Asp-Net45 previously installed. Skipping...
Checking/Installing Web-AppInit...
Web-AppInit previously installed. Skipping...
Checking/Installing Web-WebSockets...
Web-WebSockets previously installed. Skipping...
Checking/Installing Web-Mgmt-Console...
Web-Mgmt-Console previously installed. Skipping...
Checking/Installing Web-Scripting-Tools...
Web-Scripting-Tools previously installed. Skipping...
Checking/Installing Web-Mgmt-Service...
Web-Mgmt-Service previously installed. Skipping...
Checking/Installing RSAT-AD-PowerShell...
RSAT-AD-PowerShell previously installed. Skipping...
Checking/Installing Web-Request-Monitor...
Web-Request-Monitor previously installed. Skipping...
Checking/installing IisAdministrationModule...
Get-PackageSource: Unable to find repository 'NetworkShare'. Use Get-PSRepository to see all available repositories.
Done checking/installing IisAdministrationModule.
Completed feature check/install.
```

### Service Account Password, Validation, and Access Configuration

Next, you'll be prompted to enter the service account password. 

This step will add the service account as a local administrator and add logon as a batch job and as a service permissions.

> With the new PowerShell 7 console, you can use Ctrl+V to paste the password in, or simply right-click the console window and it should paste the copied password into the window. If you see `*********`, you have *something* entered as the password...

This step also sets up the deployment service accounts on the server. 

#### Ooops... Password or account issues

You messed up.

``` text
Beginning site Aibonito.jbssa.com check/creation...
Enter service account password (global\svc_sfaibonito):
WARNING: Module ActiveDirectory is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.


Checking Credentials for global\svc_sfaibonito
***************************************
Domain global.corp.prod was found: True
Get-ADUser: Variable: 'UserName' found in expression: $UserName is not defined.
Error: Username svc_sfaibonito does not exist in global.corp.prod Domain.
```

#### Already admin

The account is already set up. Nothing to see here...

``` text
Beginning site Aibonito.jbssa.com check/creation...
Enter service account password: **************
WARNING: Module ActiveDirectory is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.


Checking Credentials for global\svc_sfaibonito
***************************************
SUCCESS: The account svc_sfaibonito successfully authenticated against the domain: global.corp.prod
Checking/Adding global\svc_sfaibonito to local administrators group...
global\svc_sfaibonito is already a local administrator.
UserRightsLib.dll already decoded...
Checking/Adding service account (global\svc_sfaibonito) Logon as batch job and service permissions...
Done checking/adding service account (global\svc_sfaibonito) Logon as batch job and service permissions.
Checking/Adding GLOBAL\SVC_TFSBUILD to local administrators group...
GLOBAL\SVC_TFSBUILD is already a local administrator.
Checking/Adding GLOBAL\SVC_TFSBUILD_INT to local administrators group...
GLOBAL\SVC_TFSBUILD_INT is already a local administrator.
```

#### Adding all accounts

Woo hoo! Things are being configured...

``` text
Beginning site Aibonito.jbssa.com check/creation...
Enter service account password: **************
WARNING: Module ActiveDirectory is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.


Checking Credentials for global\svc_sfaibonito
***************************************
SUCCESS: The account svc_sfaibonito successfully authenticated against the domain: global.corp.prod
Checking/Adding global\svc_sfaibonito to local administrators group...
global\svc_sfaibonito added to local administrators group.
UserRightsLib.dll already decoded...
Checking/Adding service account (global\svc_sfaibonito) Logon as batch job and service permissions...
Done checking/adding service account (global\svc_sfaibonito) Logon as batch job and service permissions.
Checking/Adding GLOBAL\SVC_TFSBUILD to local administrators group...
GLOBAL\SVC_TFSBUILD added to local administrators group.
Checking/Adding GLOBAL\SVC_TFSBUILD_INT to local administrators group...
GLOBAL\SVC_TFSBUILD_INT added to local administrators group.
```

Now that the accounts are taken care of, you can just sit back and watch the rest fly by. The rest of this document is explaining what is being done.

### Starting with a folder

First, the folder where the website will reside is created. At the same time, the default `web.config` file is created with the redirect to the dashboard application. 

``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Creating path C:\inetpub\wwwroot\Aibonito\...

    Directory: C:\inetpub\wwwroot

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----           3/20/2023  1:03 PM                Aibonito
Creating document C:\inetpub\wwwroot\Aibonito\web.config...
```

### Creating the main app pool

Now, the main app pool is created. This will be attached to the website next.

``` text
Installing AppPool (Aibonito)...

name                        : Aibonito
queueLength                 : 1000
autoStart                   : True
enable32BitAppOnWin64       : False
managedRuntimeVersion       : v4.0
managedRuntimeLoader        : webengine4.dll
enableConfigurationOverride : True
managedPipelineMode         : Integrated
CLRConfigFile               :
passAnonymousToken          : True
startMode                   : OnDemand
state                       : Started
applicationPoolSid          : S-1-5-82-2297343673-1019498424-316720988-153919549-2122963093
processModel                : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
recycling                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
failure                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
cpu                         : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
environmentVariables        : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
workerProcesses             : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
ItemXPath                   : /system.applicationHost/applicationPools/add[@name='Aibonito']
PSPath                      : WebAdministration::\\USTXRI00NET03T\AppPools\Aibonito
PSParentPath                : WebAdministration::\\USTXRI00NET03T\AppPools
PSChildName                 : Aibonito
PSDrive                     : IIS
PSProvider                  : WebAdministration
PSIsContainer               : True
RunspaceId                  : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes                  : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement…}
ElementTagName              : add
Methods                     : {Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod}
Schema                      : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema
```

### Creating the website and attaching the app pool

Now the script will create the main website. The first step is to check that the app pool created previously is the same name as the requested website. The app pool is automatically added as part of this step.

``` text
Site name is same as new application pool name. Creating web site...

name                       : Aibonito
id                         : 15
serverAutoStart            : True
state                      : Started
bindings                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
limits                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
logFile                    : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
traceFailedRequestsLogging : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
applicationDefaults        : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
virtualDirectoryDefaults   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
ftpServer                  : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
Collection                 : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
applicationPool            : Aibonito
enabledProtocols           : http
physicalPath               : C:\inetpub\wwwroot\Aibonito
userName                   :
password                   :
ItemXPath                  : /system.applicationHost/sites/site[@name='Aibonito' and @id='15']
PSPath                     : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito
PSParentPath               : WebAdministration::\\USTXRI00NET03T\Sites
PSChildName                : Aibonito
PSProvider                 : WebAdministration
PSIsContainer              : True
RunspaceId                 : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes                 : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute}
ChildElements              : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationElement…}
ElementTagName             : site
Methods                    : {Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                             Microsoft.IIs.PowerShell.Framework.ConfigurationMethod}
Schema                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema
```

### Requesting the SSL certificate

In this step, the SSL certificate is requested, installed into the cert store, and bound to port 443.

#### The request

``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Initializing SSL for web site (Aibonito)...

Requesting SSL certificate with the following parameters:
        Common Name: Aibonitoqa.jbssa.com
        SAN: DNS=Aibonitoqa.jbssa.com,DNS=Aibonitoqa,DNS=USTXRI00NET03T,DNS=USTXRI00NET03T.global.corp.prod,DNS=10.190.2.145
        Template: NewWebServer10Year
        Country: US
        State: 
        City:
        Organization: JBS SA
        Department: IT
        CAName: cert.jbssa.com\JBS Issuing CA


Requesting SAN certificate with subject Aibonitoqa.jbssa.com and SAN: DNS=Aibonitoqa.jbssa.com&DNS=Aibonitoqa&DNS=USTXRI00NET03T&DNS=USTXRI00NET03T.global.corp.prod&DNS=10.190.2.145
Active Directory Enrollment Policy
  {5C670051-C67C-44BD-9F9F-0CBF1D9B9498}
  ldap:

CertReq: Request Created
RequestId: 92300
RequestId: "92300"
Certificate retrieved(Issued) Issued
Certificate request successfully finished!
The certificate with the subject Aibonitoqa.jbssa.com is now installed in the computer store!

```

#### Setting bindings

``` text
Binding SSL port 443...

SSL Certificate successfully added

Removing port 80...
Done initializing SSL for web site (Aibonito).
```

### API application

Here is the first of the applications to be configured. The steps are listed below.

##### Folder Creation
``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Creating path C:\inetpub\wwwroot\Aibonito\API...

    Directory: C:\inetpub\wwwroot\Aibonito

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----           3/20/2023  1:03 PM                API
```

##### App Pool Creation

``` text
Installing AppPool (Aibonito_API)...

name                        : Aibonito_API
queueLength                 : 1000
autoStart                   : True
enable32BitAppOnWin64       : False
managedRuntimeVersion       : v4.0
managedRuntimeLoader        : webengine4.dll
enableConfigurationOverride : True
managedPipelineMode         : Integrated
CLRConfigFile               :
passAnonymousToken          : True
startMode                   : OnDemand
state                       : Started
applicationPoolSid          : S-1-5-82-1539779602-53639714-3593484211-941498855-1025117090
processModel                : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
recycling                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
failure                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
cpu                         : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
environmentVariables        : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
workerProcesses             : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
ItemXPath                   : /system.applicationHost/applicationPools/add[@name='Aibonito_API']
PSPath                      : WebAdministration::\\USTXRI00NET03T\AppPools\Aibonito_API
PSParentPath                : WebAdministration::\\USTXRI00NET03T\AppPools
PSChildName                 : Aibonito_API
PSDrive                     : IIS
PSProvider                  : WebAdministration
PSIsContainer               : True
RunspaceId                  : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes                  : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement…}
ElementTagName              : add
Methods                     : {Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod}
Schema                      : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema
```

##### Service Account Setup

``` text
Configuring AppPool (Aibonito_API) to use service account login...
Configuring AppPool (Aibonito_API)...
Started
Done configuring Aibonito_API AppPool.
```

##### Web application Creation

``` text
Creating web application (API)...

path                     : /API
applicationPool          : Aibonito_API
enabledProtocols         : http
serviceAutoStartEnabled  : False
serviceAutoStartProvider :
preloadEnabled           : False
virtualDirectoryDefaults : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
Collection               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
userName                 :
password                 :
ItemXPath                : /system.applicationHost/sites/site[@name='Aibonito' and @id='15']/application[@path='/API']
Name                     : API
PSPath                   : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito\API
PSParentPath             : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito
PSChildName              : API
PSDrive                  : IIS
PSProvider               : WebAdministration
PSIsContainer            : True
PhysicalPath             : C:\inetpub\wwwroot\Aibonito\API
RunspaceId               : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes               : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements            : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
ElementTagName           : application
Methods                  :
Schema                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema

Done creating web application (API).
```

### MobileAPI Application

Everything under here is the same as [API Application](#api-application)

``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Creating path C:\inetpub\wwwroot\Aibonito\MobileAPI...
d----           3/20/2023  1:03 PM                MobileAPI
Installing AppPool (Aibonito_MobileAPI)...

name                        : Aibonito_MobileAPI
queueLength                 : 1000
autoStart                   : True
enable32BitAppOnWin64       : False
managedRuntimeVersion       : v4.0
managedRuntimeLoader        : webengine4.dll
enableConfigurationOverride : True
managedPipelineMode         : Integrated
CLRConfigFile               :
passAnonymousToken          : True
startMode                   : OnDemand
state                       : Started
applicationPoolSid          : S-1-5-82-4012986573-3015911328-900366960-2454359019-1915483230
processModel                : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
recycling                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
failure                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
cpu                         : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
environmentVariables        : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
workerProcesses             : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
ItemXPath                   : /system.applicationHost/applicationPools/add[@name='Aibonito_MobileAPI']
PSPath                      : WebAdministration::\\USTXRI00NET03T\AppPools\Aibonito_MobileAPI
PSParentPath                : WebAdministration::\\USTXRI00NET03T\AppPools
PSChildName                 : Aibonito_MobileAPI
PSDrive                     : IIS
PSProvider                  : WebAdministration
PSIsContainer               : True
RunspaceId                  : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes                  : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement…}
ElementTagName              : add
Methods                     : {Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod}
Schema                      : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema

Configuring AppPool (Aibonito_MobileAPI) to use service account login...
Configuring AppPool (Aibonito_MobileAPI)...
Started
Done configuring Aibonito_MobileAPI AppPool.
Creating web application (MobileAPI)...

path                     : /MobileAPI
applicationPool          : Aibonito_MobileAPI
enabledProtocols         : http
serviceAutoStartEnabled  : False
serviceAutoStartProvider :
preloadEnabled           : False
virtualDirectoryDefaults : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
Collection               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
userName                 :
password                 :
ItemXPath                : /system.applicationHost/sites/site[@name='Aibonito' and
                           @id='15']/application[@path='/MobileAPI']
Name                     : MobileAPI
PSPath                   : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito\MobileAPI
PSParentPath             : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito
PSChildName              : MobileAPI
PSDrive                  : IIS
PSProvider               : WebAdministration
PSIsContainer            : True
PhysicalPath             : C:\inetpub\wwwroot\Aibonito\MobileAPI
RunspaceId               : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes               : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements            : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
ElementTagName           : application
Methods                  :
Schema                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema

Done creating web application (MobileAPI).
```

### Dashboard Application

Most of this is the same as [API Application](#api-application) and [MobileAPI Application](#mobileapi-application) with the exception of handling [anonymous](#removing-anonymous-authentication) and [Windows](#adding-windows-authentication) authentication.

##### Folder Creation

``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Creating path C:\inetpub\wwwroot\Aibonito\Dashboard...
d----           3/20/2023  1:03 PM                Dashboard
```

##### App Pool Creation

``` text
Installing AppPool (Aibonito_Dashboard)...

name                        : Aibonito_Dashboard
queueLength                 : 1000
autoStart                   : True
enable32BitAppOnWin64       : False
managedRuntimeVersion       : v4.0
managedRuntimeLoader        : webengine4.dll
enableConfigurationOverride : True
managedPipelineMode         : Integrated
CLRConfigFile               :
passAnonymousToken          : True
startMode                   : OnDemand
state                       : Started
applicationPoolSid          : S-1-5-82-2568369874-478177627-2977679656-787810673-1179263646
processModel                : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
recycling                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
failure                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
cpu                         : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
environmentVariables        : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
workerProcesses             : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
ItemXPath                   : /system.applicationHost/applicationPools/add[@name='Aibonito_Dashboard']
PSPath                      : WebAdministration::\\USTXRI00NET03T\AppPools\Aibonito_Dashboard
PSParentPath                : WebAdministration::\\USTXRI00NET03T\AppPools
PSChildName                 : Aibonito_Dashboard
PSDrive                     : IIS
PSProvider                  : WebAdministration
PSIsContainer               : True
RunspaceId                  : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes                  : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement…}
ElementTagName              : add
Methods                     : {Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod}
Schema                      : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema
```

##### Service Account Setup

``` text
Configuring AppPool (Aibonito_Dashboard) to use service account login...
Configuring AppPool (Aibonito_Dashboard)...
Started
Done configuring Aibonito_Dashboard AppPool.
```

##### Web Application Creation

``` text
Creating web application (Dashboard)...

path                     : /Dashboard
applicationPool          : Aibonito_Dashboard
enabledProtocols         : http
serviceAutoStartEnabled  : False
serviceAutoStartProvider :
preloadEnabled           : False
virtualDirectoryDefaults : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
Collection               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
userName                 :
password                 :
ItemXPath                : /system.applicationHost/sites/site[@name='Aibonito' and
                           @id='15']/application[@path='/Dashboard']
Name                     : Dashboard
PSPath                   : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito\Dashboard
PSParentPath             : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito
PSChildName              : Dashboard
PSDrive                  : IIS
PSProvider               : WebAdministration
PSIsContainer            : True
PhysicalPath             : C:\inetpub\wwwroot\Aibonito\Dashboard
RunspaceId               : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes               : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements            : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
ElementTagName           : application
Methods                  :
Schema                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema

Configuring web application (Dashboard)...
Done configuring web application (Dashboard).
Done creating web application (Dashboard).
```

##### Removing Anonymous Authentication

``` text
ServerManager         : Microsoft.Web.Administration.ServerManager
IsLocked              : False
OverrideMode          : Inherit
OverrideModeEffective : Deny
SectionPath           : system.webServer/security/authentication/anonymousAuthentication
Attributes            : {enabled, userName, password, logonMethod}
ChildElements         : {}
ElementTagName        : system.webServer/security/authentication/anonymousAuthentication
IsLocallyStored       : True
Methods               :
RawAttributes         : {[enabled, True], [userName, IUSR], [password, ], [logonMethod, 3]}
Schema                : Microsoft.Web.Administration.ConfigurationElementSchema

Setting anonymousAuthentication for Aibonito/Dashboard...
Done setting anonymousAuthentication for Aibonito/Dashboard .
```

##### Adding Windows Authentication

``` text
ServerManager         : Microsoft.Web.Administration.ServerManager
IsLocked              : False
OverrideMode          : Inherit
OverrideModeEffective : Deny
SectionPath           : system.webServer/security/authentication/windowsAuthentication
Attributes            : {enabled, authPersistSingleRequest, authPersistNonNTLM, useKernelMode…}
ChildElements         : {providers, extendedProtection}
ElementTagName        : system.webServer/security/authentication/windowsAuthentication
IsLocallyStored       : True
Methods               :
RawAttributes         : {[enabled, False], [authPersistSingleRequest, False], [authPersistNonNTLM, True],
                        [useKernelMode, True]…}
Schema                : Microsoft.Web.Administration.ConfigurationElementSchema

Setting windowsAuthentication for Aibonito/Dashboard...
Done setting windowsAuthentication for Aibonito/Dashboard .
```

### DataTransfer Application

This is similar to [API Application](#api-application) and [MobileAPI Application](#mobileapi-application).

``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Creating path C:\inetpub\wwwroot\Aibonito\DataTransfer...
d----           3/20/2023  1:03 PM                DataTransfer
Installing AppPool (Aibonito_DataTransfer)...

name                        : Aibonito_DataTransfer
queueLength                 : 1000
autoStart                   : True
enable32BitAppOnWin64       : False
managedRuntimeVersion       : v4.0
managedRuntimeLoader        : webengine4.dll
enableConfigurationOverride : True
managedPipelineMode         : Integrated
CLRConfigFile               :
passAnonymousToken          : True
startMode                   : OnDemand
state                       : Started
applicationPoolSid          : S-1-5-82-1363247468-3299767607-3973473709-2746743690-3478779910
processModel                : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
recycling                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
failure                     : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
cpu                         : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
environmentVariables        : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
workerProcesses             : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
ItemXPath                   : /system.applicationHost/applicationPools/add[@name='Aibonito_DataTransfer']
PSPath                      : WebAdministration::\\USTXRI00NET03T\AppPools\Aibonito_DataTransfer
PSParentPath                : WebAdministration::\\USTXRI00NET03T\AppPools
PSChildName                 : Aibonito_DataTransfer
PSDrive                     : IIS
PSProvider                  : WebAdministration
PSIsContainer               : True
RunspaceId                  : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes                  : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationElement…}
ElementTagName              : add
Methods                     : {Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod,
                              Microsoft.IIs.PowerShell.Framework.ConfigurationMethod}
Schema                      : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema

Configuring AppPool (Aibonito_DataTransfer) to use service account login...
Configuring AppPool (Aibonito_DataTransfer)...
Started
Done configuring Aibonito_DataTransfer AppPool.
Creating web application (DataTransfer)...

path                     : /DataTransfer
applicationPool          : Aibonito_DataTransfer
enabledProtocols         : http
serviceAutoStartEnabled  : False
serviceAutoStartProvider :
preloadEnabled           : False
virtualDirectoryDefaults : Microsoft.IIs.PowerShell.Framework.ConfigurationElement
Collection               : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
userName                 :
password                 :
ItemXPath                : /system.applicationHost/sites/site[@name='Aibonito' and
                           @id='15']/application[@path='/DataTransfer']
Name                     : DataTransfer
PSPath                   : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito\DataTransfer
PSParentPath             : WebAdministration::\\USTXRI00NET03T\Sites\Aibonito
PSChildName              : DataTransfer
PSDrive                  : IIS
PSProvider               : WebAdministration
PSIsContainer            : True
PhysicalPath             : C:\inetpub\wwwroot\Aibonito\DataTransfer
RunspaceId               : 3aee01ce-7ac9-40b5-bd97-5b1b908b3a1b
Attributes               : {Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute,
                           Microsoft.IIs.PowerShell.Framework.ConfigurationAttribute…}
ChildElements            : {Microsoft.IIs.PowerShell.Framework.ConfigurationElement}
ElementTagName           : application
Methods                  :
Schema                   : Microsoft.IIs.PowerShell.Framework.ConfigurationElementSchema

Done creating web application (DataTransfer).
```

### Legacy DeploymentAgent folder

A folder to handle the storage of the legacy scale app deployment agent is needed. This does not get a website associated with it.

``` text
WARNING: Module WebAdministration is loaded in Windows PowerShell using WinPSCompatSession remoting session; please note that all input and output of commands from this module will be deserialized objects. If you want to load this module into PowerShell please use 'Import-Module -SkipEditionCheck' syntax.
Creating path C:\inetpub\wwwroot\Aibonito\DeploymentAgent\...
d----           3/20/2023  1:03 PM                DeploymentAgent
```
