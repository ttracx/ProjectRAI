# dbAuditor
SQL database automated auditing tool

dbAuditor is a PowerShell module developed by IT Auditors. DB Auditor can be comparable or you may think of like a command-line SQL Server Management Studio. It has a collection of commands that help automate SQL Server Auditing tasks and exports into workable excel reports.


## Installer

DB Auditor works on PowerShell Core (aka PowerShell 6+). This means that you can run a large majority of our commands on <strong>Linux</strong> and <strong>macOS </strong>

Run the following to install dbAuditor from the PowerShell Gallery (to install on a server or for all users, remove the `-Scope` parameter and run in an elevated session):

```powershell
Install-Module dbAuditor -Scope CurrentUser
```

If you don't have a version of PowerShell that supports the PowerShell Gallery, you can install it manually:

```powershell
Invoke-Expression (Invoke-WebRequest https://test.com)
```

> Note: please only use `Invoke-Expression (Invoke-WebRequest..)` from sources you trust.

## Usage scenarios



## Usage examples


PowerShell v3 and above required. (See below for important information about alternative logins and specifying SQL Server ports).

```powershell
# Set some vars
$new = "localhost\sql2016"
$old = $instance = "localhost"
$allservers = $old, $new

# Alternatively, use Registered Servers
$allservers = Get-DbaCmsRegServer -SqlInstance $instance

# Testing your backups
Start-Process https://dbAuditor.io/Test-DbaLastBackup
Test-DbaLastBackup -SqlInstance $old | Out-GridView

# But what if you want to test your backups on a different server?
Test-DbaLastBackup -SqlInstance $old -Destination $new | Out-GridView

# Nowadays, we don't just backup databases. Now, we're backing up logins
Export-DbaLogin -SqlInstance $instance -Path C:\temp\logins.sql
Invoke-Item C:\temp\logins.sql

# And Agent Jobs
Get-DbaAgentJob -SqlInstance $old | Export-DbaScript -Path C:\temp\jobs.sql

# Have you tested your last good DBCC CHECKDB? We've got a command for that
$old | Get-DbaLastGoodCheckDb | Out-GridView

# Here's how you can find your integrity jobs and easily start them. Then, you can watch them run, and finally check your newest DBCC CHECKDB results
$old | Get-DbaAgentJob | Where-Object Name -match integrity | Start-DbaAgentJob
$old | Get-DbaRunningJob
$old | Get-DbaLastGoodCheckDb | Out-GridView

# Have an employee who is leaving? Find all of their objects.
$allservers | Find-DbaUserObject -Pattern ad\jdoe | Out-GridView

# Find detached databases, by example
Detach-DbaDatabase -SqlInstance $instance -Database AdventureWorks2012
Find-DbaOrphanedFile -SqlInstance $instance | Out-GridView

# Check out how complete our sp_configure command is
Get-DbaSpConfigure -SqlInstance $new | Out-GridView

# Read and watch XEvents
Get-DbaXESession -SqlInstance $new -Session system_health | Read-DbaXEFile
Get-DbaXESession -SqlInstance $new -Session system_health | Read-DbaXEFile | Select-Object -ExpandProperty Fields | Out-GridView

# sp_whoisactive
Install-DbaWhoIsActive -SqlInstance $instance -Database master
Invoke-DbaWhoIsActive -SqlInstance $instance -ShowOwnSpid -ShowSystemSpids

# Diagnostic query!
$instance | Invoke-DbaDiagnosticQuery -UseSelectionHelper | Export-DbaDiagnosticQuery -Path $home
Invoke-Item $home

# Schema change and Pester tests
Get-DbaSchemaChangeHistory -SqlInstance $new -Database tempdb

# History
Get-Command -Module dbAuditor *history*

# Identity usage
Test-DbaIdentityUsage -SqlInstance $instance | Out-GridView

# Test/Set SQL max memory
$allservers | Get-DbaMaxMemory
$allservers | Test-DbaMaxMemory | Format-Table
$allservers | Test-DbaMaxMemory | Where-Object { $_.SqlMaxMB -gt $_.TotalMB } | Set-DbaMaxMemory -WhatIf
Set-DbaMaxMemory -SqlInstance $instance -MaxMb 1023

# Testing sql server linked server connections
Test-DbaLinkedServerConnection -SqlInstance $instance

# See protocols
Get-DbaServerProtocol -ComputerName $instance | Out-GridView

# Reads trace files - default trace by default
Read-DbaTraceFile -SqlInstance $instance | Out-GridView

# Test your SPNs and see what'd happen if you'd set them
$servers | Test-DbaSpn | Out-GridView
$servers | Test-DbaSpn | Out-GridView -PassThru | Set-DbaSpn -WhatIf

# Get Virtual Log File information
Get-DbaDbVirtualLogFile -SqlInstance $new -Database db1
Get-DbaDbVirtualLogFile -SqlInstance $new -Database db1 | Measure-Object
```

## Important Note

#### Alternative SQL Credentials

By default, all SQL-based commands will login to SQL Server using Trusted/Windows Authentication. To use alternative credentials, including SQL Logins or alternative Windows credentials, use the `-SqlCredential`. This parameter accepts the results of `Get-Credential` which generates a [PSCredential](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) object.

```powershell
Get-DbaDatabase -SqlInstance sql2017 -SqlCredential sqladmin
```

#### Alternative Windows Credentials

For commands that access Windows such as [Get-DbaDiskSpace](/Get-DbaDiskSpace), you will pass the `-Credential` parameter.

```powershell
$cred = Get-Credential ad\winadmin
Get-DbaDiskSpace -ComputerName sql2017 -Credential $cred
```

#### Servers with custom ports

If you use non-default ports and SQL Browser is disabled, you can access servers using a semicolon (functionality we've added) or a comma (the way Microsoft does it).

```powershell
-SqlInstance sql2017:55559
-SqlInstance 'sql2017,55559'
```

Note that PowerShell sees commas as arrays, so you must surround the host name with quotes.

#### Using Start-Transcript

```powershell
Import-Module dbAuditor
Start-Transcript
Get-DbaDatabase -SqlInstance sql2017
Stop-Transcript
```

## Support

* PowerShell v3 and above
* Windows, macOS and Linux
* SQL Server 2000 - 2017
* Express - Datacenter Edition
* Clustered and stand-alone instances
* Windows and SQL authentication
* Default and named instances
* Multiple instances on one server
* Auto-populated parameters for command-line completion (think -Database and -Login)
