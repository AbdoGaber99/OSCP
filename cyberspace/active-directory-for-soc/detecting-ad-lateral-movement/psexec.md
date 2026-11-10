# PsExec



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

### Investigating PsExec Activity

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

### sysmon indicator

1. registry value set (event code 13)

Start mode

> `TargetObject: HKLM\System\CurrentControlSet\Services\PSEXESVC\Start`

<figure><img src="../../../.gitbook/assets/001 (6).png" alt=""><figcaption></figcaption></figure>

image Path

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 211004.png" alt=""><figcaption></figcaption></figure>

2. process creation (event ID 1)

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 211909.png" alt=""><figcaption></figcaption></figure>

3. authentication log (Event ID 4648 or 4624)

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 213250.png" alt=""><figcaption></figcaption></figure>

in this log show a process creation `cmd` and parent process `PsExec`

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 213734.png" alt=""><figcaption></figcaption></figure>

***

## prevention

* &#x20;rule&#x20;

```kql
event.code: (13 or 1 or 4624)
and (process.pe.original_file_name: "psexesvc.exe" or registry.path:*Services*)
```

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-25 010301.png" alt=""><figcaption></figcaption></figure>
