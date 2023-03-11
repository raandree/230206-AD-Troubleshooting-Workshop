# 230206-AD-Troubleshooting-Workshop

- [230206-AD-Troubleshooting-Workshop](#230206-ad-troubleshooting-workshop)
  - [Labs](#labs)
  - [Useful Tools](#useful-tools)
    - [AutomatdLab](#automatdlab)
    - [ActiveDirectoryManagementFramework (ADMF)](#activedirectorymanagementframework-admf)
    - [LdapTools](#ldaptools)
    - [GPOTools](#gpotools)
    - [NTFSSecurity](#ntfssecurity)
    - [Kerberos.NET](#kerberosnet)
    - [dbatools – the community's sql powershell module](#dbatools--the-communitys-sql-powershell-module)
  - [AD Topics](#ad-topics)
    - [Active Directory Recovery Execution Service (ADRES)](#active-directory-recovery-execution-service-adres)
    - [LMCompatibilityLevel - The Most Misunderstood Windows Security Setting of All Time](#lmcompatibilitylevel---the-most-misunderstood-windows-security-setting-of-all-time)
    - [Read Active Directory Replication Metadata](#read-active-directory-replication-metadata)
  - [PowerShell Topics](#powershell-topics)
    - [Testing software and configurations](#testing-software-and-configurations)
    - [Testing the whole forest with DCDiag](#testing-the-whole-forest-with-dcdiag)
    - [Extending Event Log entries](#extending-event-log-entries)
    - [Translating an SDDL string in an ACL](#translating-an-sddl-string-in-an-acl)
    - [Reading files and folders even if it is not allowed](#reading-files-and-folders-even-if-it-is-not-allowed)
    - [JEA - Just Enough Administration](#jea---just-enough-administration)

## Labs

The labs will be available for another five months. The PowerPoint slides can also be downloaded via the Labs.
[MS Learning Campus](https://mslearningcampus.com/). To spin up your personal lab, please use the training key shared via email.

## Useful Tools

### [AutomatdLab](https://automatedlab.org/)

[AutomatedLab](https://automatedlab.org/) enables you to setup test and lab environments on Hyper-v or Azure with multiple products or just a single VM in a very short time. There are only two requirements you need to make sure: You need the DVD ISO images and a Hyper-V host or an Azure subscription.

Please refer to the documentation [Getting started](https://automatedlab.org/en/latest/Wiki/Basic/gettingstarted/) for a quick intoduction.

### [ActiveDirectoryManagementFramework (ADMF)](https://github.com/ActiveDirectoryManagementFramework/ADMF)

Central management module to orchestrate configurations and utilize the various management components that make up the Active Directory Management Framework. This framework can also be used to manage permissions (ACLs) in Active Directory.

### [LdapTools](https://github.com/FriedrichWeinmann/LdapTools)

The LdapTools PowerShell module offers the ability to run ldap queries against Active Directory without the need for the Active Directory module or ADWS. It should offer a significantly improved performance at the loss of some comfort features.

### [GPOTools](https://github.com/FriedrichWeinmann/GPOTools)

The GPOTools module is designed to handle all things GPO. As a special focus, it tries to manage migrations, backup & restore.

### [NTFSSecurity](https://github.com/raandree/NTFSSecurity)

Managing permissions with PowerShell is only a bit easier than in VBS or the command line as there are no cmdlets for most day-to-day tasks like getting a permission report or adding permission to an item. PowerShell only offers Get-Acl and Set-Acl but everything in between getting and setting the ACL is missing. This module closes the gap.

### [Kerberos.NET](https://github.com/dotnet/Kerberos.NET)

A complete Kerberos library for (KDC and Kerberos client) built entirely in managed code without (many) OS dependencies.

### [dbatools](https://dbatools.io/) – the community's sql powershell module

The [dbatools](https://dbatools.io/) is one of the most sufficticated PowerShell modules for managing SQL Server.

## AD Topics

### [Active Directory Recovery Execution Service (ADRES)](https://download.microsoft.com/documents/australia/services/datasheets2012/Active%20Directory%20Recovery%20Execution%20Service%20(ADRES).pdf)

The ADRES offering has been developed to help your organization review common disaster recovery scenarios, determine the risks posed to your business and execute the recovery steps needed to recover from disaster. By testing these common scenarios and recovery options you are able to build a fully tested and timed disaster recovery plan which will significantly reduce the time.


### LMCompatibilityLevel - The Most Misunderstood Windows Security Setting of All Time

An interesting read about the LMCompatibilityLevel setting: [The Most Misunderstood Windows Security Setting of All Time](https://learn.microsoft.com/en-us/previous-versions/technet-magazine/cc160954(v=msdn.10)).

### Read Active Directory Replication Metadata

```powershell
$o = Get-ADObject -Identity 'CN=a036564,OU=Finland,OU=Lab Accounts,DC=a,DC=vm,DC=net'
$replData = Get-ADReplicationAttributeMetadata -Object $o -Server KerbDC2 -Properties *
$replData | Sort-Object -Property AttributeName | Format-Table -Property AttributeName, Version, LastOriginatingChangeTime, LastOriginatingChangeDirectoryServerIdentity 
```

## PowerShell Topics

### Testing software and configurations

The most powerful testing framework in PowerShell is [Pester - The ubiquitous test and mock framework for PowerShell | Pester](https://pester.dev/).

The project [infraspective : Infrastructure Testing with PowerShell](https://github.com/aldrichtr/infraspective) extends Pester to also test infrastructure settings.

### Testing the whole forest with DCDiag

`dcdiag` supports the switch `/e` which calls it sequentially on all domain controllers. To run the job in parallel, you PowerShell`s `Invoke-Command`

```powershell
$computer = Get-ADDomainController -Filter *
$result = Invoke-Command -ComputerName $computer.Name -ScriptBlock { dcdiag }
```

### Extending Event Log entries

The event log message text is a static text with placeholders. The placeholders are replaces by the string array `ReplacementStrings`. These strings can be added to the event log entry as a new property which allows much better analysis.

```powershell
$e = Get-EventLog -LogName Security -InstanceId 4624 -Newest 100
$e | Add-Member -Name Account -MemberType ScriptProperty { $this.ReplacementStrings[0] }
$e | Add-Member -Name Domain -MemberType ScriptProperty { $this.ReplacementStrings[6] }
$e | Add-Member -Name ComputerAccount -MemberType ScriptProperty { $this.ReplacementStrings[5] }
$e | Format-Table -Property TimeGenerated, Account, Domain, ComputerAccount 
```

Another sample for this is [Get-KerberosEncTypes.ps1](https://gist.github.com/raandree/b90e88133861d82deb2b6496ddb3cfc3).

### Translating an SDDL string in an ACL

The class [DirectoryObjectSecurity ](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.directoryobjectsecurity?view=net-7.0) represents a security descriptor. Security descriptors can be desribed in SDDL and the class provides by means of the method `SetSecurityDescriptorSddlForm`  a way to convert SDDL into an ACL.

```powershell
$sd = [System.Security.AccessControl.DirectorySecurity]::new()
$sd = New-Object System.Security.AccessControl.DirectoryObjectSecurity

$sd.SetSecurityDescriptorSddlForm('O:DAG:DAD:PAI(A;;LCRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)(A;;CCDCLCSWRPWPLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPLOCRRCWDWO;;;S-1-5-21-2499261487-1228662389-1809124897-519)(A;;CCDCLCSWRPWPLOCRRCWDWO;;;DA)(OA;;CR;ab721a53-1e2f-11d0-9819-00aa0040529b;;WD)(OA;CI;RPWPCR;91e647de-d96f-4b70-9557-d63ff4f3ccd8;;PS)(OA;;CR;ab721a53-1e2f-11d0-9819-00aa0040529b;;PS)(OA;;RP;037088f8-0ae1-11d2-b422-00a0c968f939;4828cc14-1437-45bc-9b07-ad6f015e5f28;RU)(OA;;RP;037088f8-0ae1-11d2-b422-00a0c968f939;bf967aba-0de6-11d0-a285-00aa003049e2;RU)(OA;;RP;4c164200-20c0-11d0-a768-00aa006e0529;bf967aba-0de6-11d0-a285-00aa003049e2;RU)(OA;;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;RU)(OA;;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;bf967aba-0de6-11d0-a285-00aa003049e2;RU)(OA;;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;RU)(OA;;LCRPLORC;;4828cc14-1437-45bc-9b07-ad6f015e5f28;RU)(OA;;LCRPLORC;;bf967aba-0de6-11d0-a285-00aa003049e2;RU)(OA;;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;bf967aba-0de6-11d0-a285-00aa003049e2;RU)(OA;;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;RU)(OA;;RP;4c164200-20c0-11d0-a768-00aa006e0529;4828cc14-1437-45bc-9b07-ad6f015e5f28;RU)(OA;;RP;46a9b11d-60ae-405a-b7e8-ff8a58d456d2;;S-1-5-32-560)(OA;;RPWP;6db69a1c-9422-11d1-aebd-0000f80367c1;;S-1-5-32-561)(OA;;RPWP;5805bc62-bdc9-4428-a5e2-856a0f4c185e;;S-1-5-32-561)(OA;;RPWP;bf967a7f-0de6-11d0-a285-00aa003049e2;;CA)')

$sd.GetAccessRules($true, $true, [System.Security.Principal.NTAccount])
```

### Reading files and folders even if it is not allowed

The module NTFSSecurity provides the cmdlet `Enable-Privileges`. This cmdlet tries to enable the privileges backup, restore, security and take ownership. For being able reading all data on a computer, the backup privilege is needed.

The following command enables inheritance on all items.

```powershell
dir | Get-NTFSInheritance | Where-Object { -not $_.AccessInheritanceEnabled } | Enable-NTFSAccessInheritance
```

### JEA - Just Enough Administration

[Just Enough Administration (JEA)](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview?view=powershell-7.3) is a security technology that enables delegated administration for anything managed by PowerShell. With JEA, you can:

- Reduce the number of administrators on your machines using virtual accounts or group-managed service accounts to perform privileged actions on behalf of regular users.
- Limit what users can do by specifying which cmdlets, functions, and external commands they can run.
- Better understand what your users are doing with transcripts and logs that show you exactly which commands a user executed during their session.

The script [New-JeaDemoInDomain.ps1](https://gist.github.com/raandree/f75ebdc585013017fd3f1a5d900e0210) registers JEA endpoint which publishes a set of functions. Access to these functions is assigned via the roles "UserManagement", "UserInfo" and "DnsManagement", which are assigned to the respective group in Active Directory.

For a local demo without Active Directory, have a look at [New-RestrictedPSSessionConfigurationLocalAccounts.ps1](https://gist.github.com/raandree/ca7dc4dfafbbc8f36b1b700310bc1b5e)

The script [Connect-JeaSession.ps1](https://gist.github.com/raandree/54fd682980ffa3bdac67abf456d70f13) is for connecting to the endpoint from a client machine.
