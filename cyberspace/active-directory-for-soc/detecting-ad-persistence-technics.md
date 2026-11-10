# Detecting AD persistence Technics

#### **startup / Boot persistence**

* Startup Folder

for specific user

```
C:\Users\<Username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

for all users

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```

* Scheduled Tasks (on startup)
* service

***

#### Service persistence

1. Start Type = Automatic (start at boot)
2. Delayed Automatic (after boot)
3. Manual (if needed)



***

#### Scheduled task persistence

* on startup

```powershell
schtasks /create /tn "MyTask" /tr "C:\malware.exe" /sc onstart /ru SYSTEM
```

* onlogon

```powershell
schtasks /create /tn "UpdateService" /tr "C:\Windows\update.exe" /sc onlogon /ru SYSTEM
```

* daily / Time-based

```powershell
schtasks /create /tn "UpdateService" /tr "C:\Windows\update.exe" /sc daily /st 10:00 /ru SYSTEM
```

***

#### Registry Key persistence

* Run key

```powershell
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Updater /t REG_SZ /d "C:\malware.exe" /f
```

* RunOnce key

```powershell
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce" /v Setup /t REG_SZ /d "C:\malware.exe" /f
```

* Services&#x20;

```powershell
reg add "HKLM\System\CurrentControlSet\Services\MyService" /v ImagePath /t REG_SZ /d "C:\Windows\evil.exe" /f
reg add "HKLM\System\CurrentControlSet\Services\MyService" /v Start /t REG_DWORD /d 2 /f
```

* Winlogon

shell persistence

```powershell
reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Shell /t REG_SZ /d "explorer.exe, C:\evil.exe" /f
```

Userinit persistence

```powershell
reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /t REG_SZ /d "C:\evil.exe," /f
```

***

### Detecting Discovery With Sysmon and PowerShell Logs

```splunk
index=win EventCode=1
| search CommandLine IN ("*nltest*", "*net * user*", "*net * group*", "*net * view*", "*net * localgroup*")
| table _time, host, User, Image, CommandLine, ParentImage
| sort _time
```

```splunk
index=win EventCode=4104
| search Message IN ("*Get-ADUser*", "*Get-ADGroupMember*", "*Get-ADComputer*")
| table _time, Message
| sort _time
```
