# PowerShell Remoting

## Detection&#x20;

{% hint style="info" %}
PC2  ---> WinRM ---> DC01
{% endhint %}

### Logs in PC1&#x20;

1. PowerShell Operational Log `Microsoft-Windows-PowerShell/operation` ( New-PSSession)

<figure><img src="../../../.gitbook/assets/03 (19).png" alt=""><figcaption></figcaption></figure>

1. Authentication Logs (  Kerberos (Event ID 4624) Logon Type = 3)

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-21 233418.png" alt=""><figcaption></figcaption></figure>

1. WinRM Client Logs (Event ID 6)  `(Microsoft-Windows-WinRM/Operational)`
   * connection established
   * session created
   * target = DC1

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-21 234147.png" alt=""><figcaption></figcaption></figure>

1. Network Activity TCP connection (HTTP on Port 5985 ) (HTTPS on port 5986)

<figure><img src="../../../.gitbook/assets/02 (23).png" alt=""><figcaption></figcaption></figure>

### Logs in DC01

1. Process Creation : wsmprovhost.exe `C:\Windows\System32\wsmprovhost.exe -Embedding`  open PowerShell session

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-21 235844.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-22 000148.png" alt=""><figcaption></figcaption></figure>

run whoami after access

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-22 000219.png" alt=""><figcaption></figcaption></figure>

1. Security Logs (Event ID 4624) (Logon Type = 3 (network logon))
2. WinRM Service Logs
   * incoming session request
   * authentication result
   * session start/stop

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-22 002410.png" alt=""><figcaption></figcaption></figure>

## Prevention

1. PowerShell Remoting session

```
$cred = Get-Credential
New-PSSession -ComputerName localhost -Credential $cred
---------------------------------------------------------
Enter-PSSession -ComputerName <Name>
```

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-22 002743.png" alt=""><figcaption></figcaption></figure>

2. Get  a detection rule for this case&#x20;

[https://github.com/SigmaHQ/sigma/blob/master/rules/windows/powershell/powershell\_classic/posh\_pc\_remote\_powershell\_session.yml](https://github.com/SigmaHQ/sigma/blob/master/rules/windows/powershell/powershell_classic/posh_pc_remote_powershell_session.yml)

```yaml
title: Remote PowerShell Session (PS Classic)
id: 60167e5c-84b2-4c95-a7ac-86281f27c445
related:
    - id: 96b9f619-aa91-478f-bacb-c3e50f8df575
      type: derived
status: test
description: Detects remote PowerShell sessions
references:
    - https://threathunterplaybook.com/hunts/windows/190511-RemotePwshExecution/notebook.html
author: Roberto Rodriguez @Cyb3rWard0g
date: 2019-08-10
modified: 2024-01-03
tags:
    - attack.execution
    - attack.t1059.001
    - attack.lateral-movement
    - attack.t1021.006
logsource:
    product: windows
    category: ps_classic_start
detection:
    selection:
        Data|contains|all:
            - 'HostName=ServerRemoteHost'
            - 'wsmprovhost.exe'
    condition: selection
falsepositives:
    - Legitimate use remote PowerShell sessions
# Note: Increase the level to "medium" in environments that do not leverage PowerShell remoting
level: low
```

```
HostName=ServerRemoteHost   ---> Indicator Remote Host
wsmprovhost.exe             ---> start when PowerShell Remoting session
```

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-20 233122.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-20 233303.png" alt=""><figcaption></figcaption></figure>
