# Detecting AD Lateral Movement

## Detecting Discovery With Sysmon and PowerShell Logs

1. Sysmon

```splunk
index=win EventCode=1
| search CommandLine IN ("*nltest*", "*net * user*", "*net * group*", "*net * view*", "*net * localgroup*")
| table _time, host, User, Image, CommandLine, ParentImage
| sort _time
```

2. powershell

```splunk
index=win EventCode=4104
| search Message IN ("*Get-ADUser*", "*Get-ADGroupMember*", "*Get-ADComputer*")
| table _time, Message
| sort _time
```

## How Lateral movement

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

#### Logon Type: The First Thing to Check

<table><thead><tr><th width="109">Logon Type</th><th width="174">Meaning</th><th width="136">Common Protocol</th><th>What It Tells Us</th></tr></thead><tbody><tr><td>3</td><td>Network logon</td><td>SMB, PsExec</td><td>Remote access without an interactive session</td></tr><tr><td>7</td><td>Unlock/Reconnect</td><td>RDP</td><td>Session reconnect or workstation unlock</td></tr><tr><td>10</td><td>RemoteInteractive</td><td>RDP</td><td>Full desktop session</td></tr></tbody></table>

## Detect SMB lateral movement

#### SMB Authentication

**User authentication** - Users must provide a username and password in order to authenticate with the SMB server in order to access a share.&#x20;

**Share authentication** - Users must provide a password in order to access restricted share.

* Enable Audit file share

<figure><img src="../../../.gitbook/assets/Screenshot 2026-07-30 123414.png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Event code: 5140  `A network share object was accessed`

Event code: 4624 `Logon Type = 3`


{% endhint %}

#### Admin Shares

* **Investigating Event 5140 File Share Access**

```splunk
index=win EventCode=5140 Share_Name IN ("*\\ADMIN$\*", "*\\C$\*")
| table _time, host, Source_Address, user, Share_Name
| sort _time
```

***

## Detect PsExec

{% hint style="danger" %}
Event code : 4624 `Authentication`

Event code : 5140 `Connect to ADMIN$ share`

Event code : 7045 `create new service PSEXESVC`&#x20;

* **Service Type = User Mode Service**
* **Start Type = Demand Start**
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

## Detection&#x20;

```
./PsExec64.exe -i \\FILESRV01 -u demp\abdo.gaber -p password cmd
```

#### Investigating PsExec Activity

```splunk
index=win EventCode=7045
| table _time, host, Service_Name, Service_File_Name, Service_Type, Service_Start_Type, Service_Account
| sort _time

```

* sysmon event 1

```splunk
index=win EventCode=1 host={DESTINATION_HOST} ParentImage="*PSEXESVC*"
| table _time, host, User, ParentImage, Image, CommandLine
| sort _time
```

* sysmon event 17 (Named Pipes)

```splunk
index=win EventCode=17 Image="*PSEXESVC*"
| table _time, host, Image, PipeName
| sort _time
```

## Detect RDP

{% hint style="danger" %}
Event code = 4624

* Logon Type = 10

`mstsc.exe` (the Windows Remote Desktop Client) runs on the target machine
{% endhint %}

#### Investigating RDP Lateral Movement&#x20;

* target machine

```splunk
index=win EventCode=1 host=THM-DC
| search CommandLine IN ("*nltest*", "*net * user*", "*net * group*", "*net * view*")
| table _time, host, User, Image, CommandLine, LogonId
| sort _time
```

```splunk
index=win EventCode=4624 host=THM-DC Logon_ID={LOGON_ID}
| table _time, user, Logon_Type, Source_Network_Address, Logon_ID
```

```splunk
index=win EventCode=1 host={SOURCE_SERVER} Image="*mstsc.exe*"
| table _time, User, Image, CommandLine, LogonId
| sort _time
```

