# Get-WinEvent

* **List all of Event Log**&#x20;

```powershell
Get-WinEvent -ListLog * | Select-Object LogName, RecordCount, IsEnabled, LogMode, LogType
```

* **the list of providers and their respective linked logs**

```powershell
Get-WinEvent -ListProvider * | Format-Table -AutoSize
```

1. **Retrieving events from the System log**

```powershell
Get-WinEvent -LogName 'System' -MaxEvents 50 |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Table -AutoSize
```

2. **Retrieving events from Microsoft-Windows-WinRM/Operational**

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -MaxEvents 30 |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Ta ble -AutoSize
```

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -Oldest -MaxEvents 30 |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Table -AutoSize
```

3. **Retrieving events from .evtx Files**

```powershell
Get-WinEvent -Path 'C:\Tools\exec_sysmon_1.evtx' -MaxEvents 5 |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Table -AutoSize

```

4. **Filtering events with FilterHashtable**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    ID=1,3
} |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Table -AutoSize
```

* **Filtering events with FilterHashtable From external `.evtx`**&#x20;

```powershell
Get-WinEvent -FilterHashtable @{
    Path='C:\Tools\sysmon_mshta_sharpshooter_stageless_meterpreter.evtx'
    ID=1,3
} |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Table -AutoSize
```

* **Filter Event With Date Range**

```powershell
$startDate = (Get-Date -Year 2023 -Month 5 -Day 28).Date
$endDate   = (Get-Date -Year 2023 -Month 6 -Day 3).Date
```

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    ID=1,3
    StartTime=$startDate
    EndTime=$endDate
} |
Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message |
Format-Table -AutoSize
```

5. **Filtering events with FilterHashtable & XML**

* investigate the potential threat. where a suspicious network connection to a particular IP (`52.113.194.132`)

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    $eventData = $xml.Event.EventData.Data
    New-Object PSObject -Property @{
        SourceIP = $eventData | Where-Object {$_.Name -eq "SourceIp"} | Select-Object -ExpandProperty '#text'
        DestinationIP = $eventData | Where-Object {$_.Name -eq "DestinationIp"} | Select-Object -ExpandProperty '#text'
        ProcessGuid = $eventData | Where-Object {$_.Name -eq "ProcessGuid"} | Select-Object -ExpandProperty '#text'
        ProcessId = $eventData | Where-Object {$_.Name -eq "ProcessId"} | Select-Object -ExpandProperty '#text'
    }
} | Where-Object {$_.DestinationIP -eq "52.113.194.132"}
```

* investigate to the DLL loaded for anomalous `clr.dll` and `mscoree.dll`&#x20;

```powershell
$Query = @"
	<QueryList>
		<Query Id="0">
			<Select Path="Microsoft-Windows-Sysmon/Operational">*[System[(EventID=7)]] and *[EventData[Data='mscoree.dll']] or *[EventData[Data='clr.dll']]
			</Select>
		</Query>
	</QueryList>
	"@
```

```powershell
Get-WinEvent -FilterXml $Query | ForEach-Object {Write-Host $_.Message `n}
```

6. **Filtering events with FilterXPath**

* investigate to install Sysinternals tools

```powershell
Get-WinEvent `
 -LogName 'Microsoft-Windows-Sysmon/Operational' `
 -FilterXPath "
 *[
   EventData[
     Data[@Name='Image']='C:\Windows\System32\reg.exe'
   ]
 ]
 and
 *[
   EventData[
     Data[@Name='CommandLine']=
     '\"C:\Windows\system32\reg.exe\" ADD HKCU\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f'
   ]
 ]"
 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message
 | Format-Table -AutoSize

```

7. **Filtering events based on property values**

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} -MaxEvents 1 | Select-Object -Property *
```
