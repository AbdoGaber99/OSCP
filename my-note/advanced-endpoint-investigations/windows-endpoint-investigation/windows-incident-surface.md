---
description: Proactive Threat Hunting
---

# Windows Incident Surface

## Reliability of the System Tools

### Learning Objectives

* Verify the integrity of system tools to maintain the integrity of the investigation.
* Collect system information to identify system and user profile and uncover some indicators of anomalies.
* Read data from crucial forensic artefacts to hunt for unwanted persistent objects.
* Analyze processes and directories to find traces of suspicious activity

### <mark style="color:red;">PowerShell Profiles</mark>

$profile

```powershell
C:\Users\<User>\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

```shell-session
where powershell.exe
```

```powershell
exist "C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.PowerShell_profile.ps1" (echo PROFILE EXISTS) else (echo PROFILE DOES NOT EXIST)
```

```powershell
type "C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1"
```

#### Restoring PowerShell profile

* Rename the safe PS profile file to be used in the system

```
ren Clean_profile.ps1 profile.ps1
```

* Change the existing profile file extension to backup (.bak)

```shell-session
ren C:\Windows\System32\WindowsPowerShell\v1.0\\profile.ps1 profile.bak
```

* Move the safe profile file to the existing profile path

```shell-session
copy profile.ps1 C:\Windows\System32\WindowsPowerShell\v1.0\
```

#### Loaded Modules

modules path:

```
$env:PSModulePath
```

```shell-session
Get-Module
```

```shell-session
Get-Module -ListAvailable
```

***

### <mark style="color:red;">System Profile</mark>

#### System Name and Network Information

```powershell
Get-CimInstance win32_networkadapterconfiguration -Filter IPEnabled=TRUE | ft DNSHostname, IPAddress, MACAdrress
```

#### OS Version and Installation Details

```powershell
Get-CimInstance -ClassName Win32_OperatingSystem | fl CSName, Version, BuildNumber, InstallDate, LastBootUpTime, OSArchitecture
```

#### Date and TimeZone Info

```powershell
Get-Date ; Get-TimeZone
```

#### Reviewing Policies

```powershell
Get-GPResultantSetOfPolicy -ReportType HTML -Path (Join-Path -Path (Get-Location).Path -ChildPath "RSOPReport.html")
```

***

### <mark style="color:red;">Users and Sessions</mark>

* Users

```powershell
Get-LocalUser | Select-Object Name, Enable, LastLogon
```

```powershell
Get-CimInstance -Class Win32_UserAccount -Filter "LocalAccount=True" | Format-Table  Name, PasswordRequired, PasswordExpires, PasswordChangeable | Tee-Object "user-details.txt"
```

* Group

```powershell
Get-LocalGroup | ForEach-Object { $members = Get-LocalGroupMember -Group $_.Name; if ($members) { Write-Output "`nGroup: $($_.Name)"; $members | ForEach-Object { Write-Output "`tMember: $($_.Name)" } } }
```

* Sessions

```powershell
.\PsLoggedon64.exe
```

***

### <mark style="color:red;">Network Scope</mark>

* Active Ports and Connection

```powershell
Get-NetTCPConnection | select Local*, Remote*, State, OwningProcess,` @{n="ProcName";e={(Get-Process -Id $_.OwningProcess).ProcessName}},` @{n="ProcPath";e={(Get-Process -Id $_.OwningProcess).Path}} | sort State | ft -Auto | tee tcp-conn.txt
```

* Network share

```powershell
Get-CimInstance -Class Win32_Share | tee net-shares.txt
```

* Firewall (Network)

```powershell
Get-NetFirewallProfile | ft Name, Enabled, DefaultInboundAction, DefaultOutboundAction | tee fw-profiles.txt
```

***

### <mark style="color:red;">Startup and registry</mark>

* Startup Programs and Commands

```powershell
Get-CimInstance Win32_StartupCommand | Select-Object Name, command, Location, User | fl
```

* Boot-Time Execution Entries

```powershell
.\autorunsc64.exe -a b * -h
```

* Logon-Time Execution Entries

```powershell
.\autorunsc64.exe -a l * -h
```

* `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit`
* `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell`

Enumerating a Registry Key

```powershell
$winlogonPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"; "Userinit: $((Get-ItemProperty -Path $winlogonPath -Name 'Userinit').Userinit)"; "Shell: $((Get-ItemProperty -Path $winlogonPath -Name 'Shell').Shell)"
```

Getting Details of a Registry Key

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\NetSh"
```

***

### <mark style="color:red;">Services and Scheduled Items</mark>

* Services

Current Running Services

```powershell
"Running Services:"; Get-CimInstance -ClassName Win32_Service | Where-Object { $_.State -eq "Running" } | Select-Object Name, DisplayName, State, StartMode, PathName, ProcessId | ft -AutoSize | tee services-active.txt
```

Idle Services

```powershell
"Non-Running Services:"; Get-CimInstance -ClassName Win32_Service | Where-Object { $_.State -ne "Running" } | Select-Object @{Name='Name'; Expression={if ($_.Name.Length -gt 22) { "$($_.Name.Substring(0,19))..." } else { $_.Name }}}, @{Name='DisplayName'; Expression={if ($_.DisplayName.Length -gt 45) { "$($_.DisplayName.Substring(0,42))..." } else { $_.DisplayName }}}, State, StartMode, PathName, ProcessId | Format-Table -AutoSize | Tee-Object services-idle.txt
```

* Scheduled Tasks and Scheduled Jobs

```powershell
$tasks = Get-CimInstance -Namespace "Root/Microsoft/Windows/TaskScheduler" -ClassName MSFT_ScheduledTask; if ($tasks.Count -eq 0) { Write-Host "No scheduled tasks found."; exit } else { Write-Host "$($tasks.Count) scheduled tasks found." }; $results = @(); foreach ($task in $tasks) { foreach ($action in $task.Actions) { if ($action.PSObject.TypeNames[0] -eq 'Microsoft.Management.Infrastructure.CimInstance#Root/Microsoft/Windows/TaskScheduler/MSFT_TaskExecAction') { $results += [PSCustomObject]@{ TaskPath = $task.TaskPath.Substring(0, [Math]::Min(50, $task.TaskPath.Length)); TaskName = $task.TaskName.Substring(0, [Math]::Min(50, $task.TaskName.Length)); State = $task.State; Author = $task.Principal.UserId; Execute = $action.Execute } } } }; if ($results.Count -eq 0) { Write-Host "No tasks with 'MSFT_TaskExecAction' actions found." } else { $results | Format-Table -AutoSize | tee scheduled-tasks.txt }
```

***

### <mark style="color:red;">Processes and Directories</mark>

* processes

```powershell
Get-WmiObject -Class Win32_Process | ForEach-Object {$owner = $_.GetOwner(); [PSCustomObject]@{Name=$_.Name; PID=$_.ProcessId; P_PID=$_.ParentProcessId; User="$($owner.User)"; CommandLine=if ($_.CommandLine.Length -le 60) { $_.CommandLine } else { $_.CommandLine.Substring(0, 60) + "..." }; Path=$_.Path}} | ft -AutoSize
```

* Directories

Details of the Temp Folders

```powershell
Get-ChildItem -Path "C:\Users" -Force | Where-Object { $_.PSIsContainer } | ForEach-Object { Get-ChildItem -Path "$($_.FullName)\AppData\Local\Temp" -Recurse -Force -ErrorAction SilentlyContinue | Select-Object @{Name='User';Expression={$_.FullName.Split('\')[2]}}, FullName, Name, Extension } | ft -AutoSize
```

Further Details of a Temp Folder

```powershell
Get-ChildItem -Path "C:\Users\Administrator\AppData\" -Recurse -Force | ft FullName, Name, Extension
```

List the Disk Volumes of the System

```powershell
Get-CimInstance -ClassName Win32_Volume | ft -AutoSize DriveLetter, Label, FileSystem, Capacity, FreeSpace
```

