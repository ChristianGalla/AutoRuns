﻿AutoRuns PowerShell Module
==========================

**AutoRuns module was designed to help do live incident response and enumerate autoruns artifacts that may be used by legitimate programs as well as malware to achieve persistence.**

## Table of Contents  
* [Usage](#Usage)
  * [Install the module](#Install)
  * [Functions](#Functions)
  * [Help](#Help)
* [Issues](#issues)
* [Todo](#Todo)
* [Credits](#Credits)
* [Original Autoruns.exe release history](#AutorunsHistory)

<a name="Usage"/>

## Usage

<a name="Install"/>

### Install the module

```powershell
# Check the mmodule on powershellgallery.com
Find-Module -Name Autoruns -Repository PSGallery
```
```
Version    Name                                Repository           Description
-------    ----                                ----------           -----------                                   
13.95    AutoRuns                            PSGallery            AutoRuns is a module ...
```

```powershell
# Save the module locally in Downloads folder
Save-Module -Name AutoRuns -Repository PSGallery -Path ~/Downloads
```

Stop and please review the content of the module, I mean the code to make sure it's trustworthy :-)

You can also verify that the SHA256 hashes of downloaded files match those stored in the catalog file
```powershell
$HT = @{
    CatalogFilePath = "~/Downloads/AutoRuns/13.95/AutoRuns.cat"
    Path = "~/Downloads/AutoRuns/13.95"
    Detailed = $true
    FilesToSkip = 'PSGetModuleInfo.xml'
}
Test-FileCatalog @HT
```

```powershell
# Import the module
Import-Module ~/Downloads/AutoRuns/13.95/AutoRuns.psd1 -Force -Verbose
```

<a name="Functions"/>

### Check the command available
```powershell
Get-Command -Module AutoRuns
```
```
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Get-PSAutorun                                      13.95      AutoRuns
```


<a name="Help"/>

### Find the syntax

```powershell
# View the syntax
Get-Command Get-PSAutorun -Syntax
```
```
Get-PSAutorun [-All] [-BootExecute] [-AppinitDLLs] [-ExplorerAddons] [-SidebarGadgets] [-ImageHijacks]
[-InternetExplorerAddons] [-KnownDLLs] [-Logon] [-Winsock] [-Codecs] [-OfficeAddins] 
[-PrintMonitorDLLs] [-LSAsecurityProviders] [-ServicesAndDrivers] [-ScheduledTasks] [-Winlogon] 
[-WMI] [-ShowFileHash] [-VerifyDigitalSignature] [-User <string>] [<CommonParameters>]
```

### View examples provided in the help
```powershell
# Get examples from the help
 Get-Help Get-PSAutorun  -Examples
```
```
NAME
    Get-PSAutorun

SYNOPSIS
    Get Autorun entries.

    -------------------------- EXAMPLE 1 --------------------------

    PS C:\>Get-PSAutorun -BootExecute -AppinitDLLs

    -------------------------- EXAMPLE 2 --------------------------

    PS C:\>Get-PSAutorun -KnownDLLs -LSAsecurityProviders -ShowFileHash

    -------------------------- EXAMPLE 3 --------------------------

    PS C:\>Get-PSAutorun -All -ShowFileHash -VerifyDigitalSignature

    -------------------------- EXAMPLE 4 --------------------------

    PS C:\>Get-PSAutorun -All -User * -ShowFileHash -VerifyDigitalSignature

```

<a name="Issues"/>

## Issues
 * What are registrations in the WMI\Default namespace introduced in Autoruns v13.7? see [c7eab48c77f578e0dcff61d2b46a479b28225a56](https://github.com/p0w3rsh3ll/AutoRuns/commit/c7eab48c77f578e0dcff61d2b46a479b28225a56)

* If you run PowerShell 5.1 and Applocker in allow mode, you need to merge an appplocker rule to your local policy so that the digital signature of this module is trusted. With that in place, it will allow you to successfully load the module and use the function in an interactive shell where PowerShell language mode is set to ConstrainedLanguage.
```powershell
if ($ExecutionContext.SessionState.LanguageMode -eq 'ConstrainedLanguage') {
    Get-Content -Path AWL.xml
    Set-AppLockerPolicy -XmlPolicy AWL.xml -Verbose -Merge
}
```
If your corporate admin has turned off local group policy objects processing on a domain joined device, you'll need to add the trusted publisher rule in a Domain group policy.
```powershell
gp 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\System' -Name DisableLGPOProcessing -EA 0
```

<a name="Todo"/>

## Todo

#### Coding best practices
- [x] Use PSScriptAnalyzer module to validate the code follows best practices
- [ ] Write Pester tests for this module

#### OS and Software compatibility
- [x] Test the module in PowerShell Core 6.0.0
- [ ] Test the module on Nano and get rid of Add-Member cmdlet
- [ ] Test the module on various versions of Windows 10
- [ ] Test the module on Windows RT
- [ ] Review Office Add-ins code with Office x86 and x64 versions

#### General improvements
- [ ] Write a better implementation of the internal Get-RegValue function
- [ ] Review and improve regex used by the internal Get-PSPrettyAutorun function (ex: external paths)

#### New features
- [x] Replace HKCU and add an option to specify what user hive is being investigated
- [ ] Add timestamps on registry keys
- [ ] Analyze an offline image of Windows

#### Help
- [ ] More examples
- [ ] Use external help? 
- [ ] Internationalization?
- [ ] Copy the commetented changelog at the end of the module in README.md
- [ ] Document issues and write a pester tests to validate the module behavior if fixed

<a name="Credits"/>

## Credits
Thanks go to:
* **[@LeeHolmes](https://github.com/LeeHolmes/)**: 
    * Improving common parameters [7fe8e7ea983325b5543da1300cb8a4636c7062ef](https://github.com/p0w3rsh3ll/AutoRuns/commit/7fe8e7ea983325b5543da1300cb8a4636c7062ef)
    * Filtering OS binaries [8efb0fad4585ecd4517b55ff5b5cec91f7dd364a](https://github.com/p0w3rsh3ll/AutoRuns/commit/8efb0fad4585ecd4517b55ff5b5cec91f7dd364a)
```powershell
Get-PSAutorun -VerifyDigitalSignature | ? { -not $_.IsOSBinary }
```

<a name="AutorunsHistory"/>

## Original [Autoruns.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns) from Mark Russinovich

[Autoruns v13.95](https://blogs.technet.microsoft.com/sysinternals/2019/06/12/sysmon-v10-0-autoruns-v13-95-vmmap-v3-26/)
>This Autoruns update adds support for user Shell folders redirections.

[Autoruns v13.94](https://blogs.technet.microsoft.com/sysinternals/2019/02/19/sysmon-v9-0-autoruns-v13-94/)
>This Autoruns update fixes a bug that prevented the correct display of the target of image hosts such as svchost.exe, rundll32.exe, and cmd.exe.

[Autoruns v13.93](https://blogs.technet.microsoft.com/sysinternals/2018/12/09/autoruns-v13-93-handle-v4-21-process-explorer-v16-22-sdelete-v2-02-sigcheck-v2-71-sysmon-v8-02-and-vmmap-v3-25/)
>This Autoruns update fixes a bug that prevented UserInitMprLogonScript from being scanned and by-default enables HCKU scanning for the console version.

[Autoruns v13.90](https://blogs.technet.microsoft.com/sysinternals/2018/07/05/sysmon-v8-0-autoruns-v13-90/)
>Autoruns, a comprehensive Windows autostart entry point (ASEP) manager, now includes Runonce\*\Depend keys and GPO logon and logoff locations, as well as fixes a bug in WMI path parsing.

[Autoruns v13.82](https://blogs.technet.microsoft.com/sysinternals/2018/02/17/process-monitor-v3-50-autoruns-v13-82-du-v1-61-sdelete-v2-01/)
>This Autoruns release shows Onenote addins and fixes several bugs.

[Autoruns v13.81](https://blogs.technet.microsoft.com/sysinternals/2017/12/12/autoruns-v13-81-bginfo-v423-handle-v4-11/)
> This update to Autoruns fixes a Wow64 bug in Autorunsc that could cause 32-bit paths to result in 'file not found' errors, and expands the set of images not considered part of Windows for the Windows filter in order to reveal malicious files masquerading as Windows images

[Autoruns v13.80](https://blogs.technet.microsoft.com/sysinternals/2017/09/12/sysinternals-update-sysmon-v6-1-process-monitor-v3-4-autoruns-v13-8-accesschk-v6-11/)
> This release of Autoruns, a utility for viewing and managing autostart execution points (ASEPs), adds additional autostart entry points, has asynchronous file saving, fixes a bug parsing 32-bit paths on 64-bit Windows, shows the display name for drivers and services, and fixes a bug in offline Virus Total scanning.

[Autoruns v13.71](https://blogs.technet.microsoft.com/sysinternals/2017/05/16/sysinternals-update-procdump-v9-autoruns-v13-71-bginfo-v4-22-livekd-v5-62-process-monitor-v3-33-process-explorer-v16-21/)
> This update to Autoruns, a comprehensive autostart execution point manager, adds Microsoft HTML Application Host (mshta.exe) as hosting image so it displays the hosted image details, and now doesn’t apply filters to hosting images.

[Autoruns v13.7](https://blogs.technet.microsoft.com/sysinternals/2017/02/17/update-sysmon-v6-autoruns-v13-7-accesschk-v6-1-process-monitor-v3-32-process-explorer-v16-2-livekd-v5-61-and-bginfo-v4-21/)
> Autoruns, an autostart entry point management utility, now reports print providers, registrations in the WMI\Default namespace, fixes a KnownDLLs enumeration bug, and has improved toolbar usability on high-DPI displays.

[Autoruns v13.51](https://blogs.technet.microsoft.com/sysinternals/2016/01/05/update-sigcheck-v2-4-sysmon-v3-2-process-explorer-v16-1-autoruns-v13-51-accesschk-v6-01/)
> This release of Autoruns, a comprehensive autostart entry manager, fixes a WMI command-line parsing bug, emits a UNICODE BOM in the file generated when saving results to a text file, and adds back the ability to selectively verify the signing status of individual entries.

[Autoruns v13.5](https://blogs.technet.microsoft.com/sysinternals/2015/10/26/update-autoruns-v13-5-sigcheck-v2-3-rammap-v1-4-bginfo-v4-21-sysmon-v3-11-adinsight-v1-2/)
>This update to Autoruns, the most comprehensive autostart viewer and manager available for Windows, now shows 32-bit Office addins and font drivers, and enables resubmission of known images to Virus Total for a new scan.

[Autoruns v13.4](https://blogs.technet.microsoft.com/sysinternals/2015/05/26/update-accesschk-v6-0-autoruns-v13-4-process-monitor-v3-2-vmmap-v3-2/)
>Autoruns, the most comprehensive utility available for showing what executables, DLLs, and drivers are configured to automatically start and load, now reports Office addins, adds several additional autostart locations, and no longer hides hosting executables like cmd.exe, powershell.exe and others when Windows and Microsoft filters are in effect.

[Autoruns v13.3](https://blogs.technet.microsoft.com/sysinternals/2015/04/20/update-sysmon-v3-0-autornus-v13-3-regjump-v1-1-process-monitor-v3-11/)
>Autoruns, a utility that shows what processes, DLLs, and drivers are configured to automatically load, adds reporting of GP extension DLLs and now shows the target of hosting processes like cmd.exe and rundll32.exe.

[Autoruns v13.2](https://blogs.technet.microsoft.com/sysinternals/2015/03/10/update-livekd-v5-4-autoruns-v13-2-sigcheck-v2-2-process-explorer-v16-05/)
>In addition to bug fixes to CSV and XML output, Autorunsc introduces import-hash reporting, and Autoruns now excludes command-line and other host processes from the Microsoft and Windows filters.

[Autoruns v13.01](https://blogs.technet.microsoft.com/sysinternals/2015/02/09/update-autoruns-v13-01/)
>This release fixes a bug in v13 that caused autostart entry lines not to show when you enter a filter string into the toolbar's filter control

[Autoruns v13.0](https://blogs.technet.microsoft.com/sysinternals/2015/01/29/update-autoruns-v13-0/)
> This major update to Autoruns, an autostart execution point (ASEP) manager, now has integration with Virustotal.com to show the status of entries with respect to scans by over four dozen antimalware engines. It also includes a revamped scanning architecture that supports dynamic filters, including a free-form text filter, a greatly improved compare feature that highlights not just new items but deleted ones as well, and file saving and loading that preserves all the information of a scan

[Autoruns v12.03](https://blogs.technet.microsoft.com/sysinternals/2014/09/11/updates-handle-v4-0-procdump-v7-01-procexp-v16-04-regjump-v1-02-autoruns-v12-03/)
>This update to Autoruns adds the registered HTML file extension, fixes a bug that could cause disabling of specific entry types to fail with a “path not found” error, and addresses another that could prevent the Jump-to-image function from opening the selected image on 64-bit Windows.
 
[Autoruns v12.02](https://blogs.technet.microsoft.com/sysinternals/2014/08/19/updates-autoruns-v12-02-coreinfo-v3-31-sysmon-v1-01-whois-v1-12/)
>This fixes a bug that could cause Autoruns to crash on startup, updates the image path parsing for Installed Components to remove false positive file-not-found entries, and correctly reports image entry timestamps in local time instead of UTC.

[Autoruns v12.01](https://blogs.technet.microsoft.com/sysinternals/2014/08/08/new-sysmon-v1-0-updates-autoruns-v12-01-coreinfo-v3-3-procexp-v16-03/)
>This update to Autoruns, a utility that comes in Windows application and command-line forms, has numerous bug fixes, adds a profile attribute/column to CSV and XML output, and interprets the CodeBase value for COM object registrations.

[Autoruns v12.0](https://blogs.technet.microsoft.com/sysinternals/2014/05/13/updates-autoruns-v12-0-procdump-v7-0/)
>This release of Autoruns, a Windows application and command-line utility for viewing autostart entries, now reports the presence of batch file and executable image entries in the WMI database, a vector used by some types of malware.
 
[Autoruns v11.70](https://blogs.technet.microsoft.com/sysinternals/2013/08/01/autoruns-v11-70-bginfo-v4-20-disk2vhd-v1-64-process-explorer-v15-40/)
>This release of Autoruns, a powerful utility for scanning and disabling autostart code, adds a new option to have it show only per-user locations, something that is useful when analyzing the autostarts of different accounts than the one that Autoruns is running under.

[Autoruns v11.62](https://blogs.technet.microsoft.com/sysinternals/2013/07/01/update-autoruns-v11-62/)
>This release fixes a bug in version 11.61’s jump-to-image functionality.