# Detecting AD Credential Attacks

## Detecting Kerberoasting&#x20;

{% hint style="danger" %}
monitor&#x20;

* event code : 4769
* Encryption type : RC4 (0x17)
{% endhint %}

#### How Do Accounts End Up Kerberoastable?

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

#### Detection with Splunk

```spl
index=task2 EventCode=4769 Ticket_Encryption_Type=0x17 Service_Name!="*$" Service_Name!="krbtgt"
| table _time, Account_Name, Service_Name, Ticket_Encryption_Type, Client_Address
| sort _time
```

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

***

## Detecting AS-REP Roasting

{% hint style="danger" %}
monitor&#x20;

* event code : 4768
* Pre\_Authentication\_Type: 0
{% endhint %}

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

#### Detection with Splunk

* all TGT&#x20;

```splunk
index=task3 EventCode=4768
| table _time, Account_Name, Pre_Authentication_Type, Ticket_Encryption_Type, Client_Address
| sort _time
```

* TGT with Pre\_Authentication

```splunk
index=task3 EventCode=4768 Pre_Authentication_Type=0
| table _time, Account_Name, Pre_Authentication_Type, Ticket_Encryption_Type, Client_Address
```

#### Kerberoasting vs AS-REP Roasting

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

***

## Detecting LSASS Credential dumping

{% hint style="danger" %}
sysmon Event code : 10
{% endhint %}

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

#### Here are the fields we'll use for detection:

<table><thead><tr><th width="147">Field</th><th>What It Contains</th><th>Why It Matters</th></tr></thead><tbody><tr><td>SourceImage</td><td>Full path of the process accessing LSASS</td><td>Identifies the tool used for the dump</td></tr><tr><td>SourceUser</td><td>The user account running the source process</td><td>Identifies the compromised account. SYSTEM is expected; a domain user account is suspicious</td></tr><tr><td>TargetImage</td><td>Full path of the target process (lsass.exe)</td><td>Confirms LSASS was the target</td></tr><tr><td>GrantedAccess</td><td>Hex access mask showing requested permissions</td><td>Different tools request different access levels</td></tr><tr><td>CallTrace</td><td>DLL call stack leading to the access</td><td>Reveals the method used (MiniDump API, injection, etc.)</td></tr></tbody></table>

#### Understanding the GrantedAccess Field

<table><thead><tr><th>Access Right</th><th width="133">Hex Value</th><th>Purpose</th></tr></thead><tbody><tr><td>PROCESS_QUERY_LIMITED_INFORMATION</td><td>0x1000</td><td>Query basic process info</td></tr><tr><td>PROCESS_QUERY_INFORMATION</td><td>0x0400</td><td>Query detailed process info</td></tr><tr><td>PROCESS_VM_READ</td><td>0x0010</td><td>Read process memory</td></tr><tr><td>PROCESS_ALL_ACCESS</td><td>0x1FFFFF</td><td>Full access to the process</td></tr></tbody></table>

#### Understanding the CallTrace Field

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

#### Normal case

<table><thead><tr><th width="300">ull Process Path</th><th width="90">Typical GrantedAccess</th><th>Why</th></tr></thead><tbody><tr><td>C:\Windows\System32\csrss.exe</td><td>0x1000 or 0x1400</td><td>Windows subsystem, manages processes</td></tr><tr><td>C:\Windows\System32\WerFault.exe</td><td>0x1000</td><td>Windows Error Reporting crash handler</td></tr><tr><td>C:\Windows\System32\svchost.exe</td><td>0x1010</td><td>Various service functions (normal at this access level)</td></tr><tr><td>AV/EDR agent paths</td><td>Varies</td><td>Security products monitor LSASS for protection</td></tr></tbody></table>

#### Detection with Splunk

```splunk
index=task4 EventCode=10 TargetImage="*\\lsass.exe"
| stats count by SourceImage, GrantedAccess
```

***

## Detecting  DCSync

{% hint style="danger" %}
monitor&#x20;

Event code: 4662

GUID of Extended Rights permission: `1131f6ad`
{% endhint %}

#### Extended Rights permission

* `{1131f6ad-9c07-11d1-f79f-00c04fc2dcd2}` is DS-Replication-Get-Changes-All
* `{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}` is DS-Replication-Get-Changes
* `{89e95b76-444d-4c62-991a-0facbeda640c}` is DS-Replication-Get-Changes-In-Filtered-Set

#### Detection with Event 4662

<table><thead><tr><th width="137">Field</th><th>What It Contains</th><th>Why It Matters</th></tr></thead><tbody><tr><td>user</td><td>The account performing the replication</td><td>Identifies who is running DCSync</td></tr><tr><td>Access_Mask</td><td>0x100 (Control Access)</td><td>Indicates an extended right was exercised</td></tr><tr><td>Properties</td><td>Shows "Control Access" (GUIDs in raw event)</td><td>The replication GUIDs confirm DCSync</td></tr><tr><td>Logon_ID</td><td>Hex session identifier</td><td>Links to 4624 logon event for source IP correlation</td></tr></tbody></table>

#### Normal vs Suspicious Replication

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<table><thead><tr><th width="130">Pattern</th><th>Normal</th><th>Suspicious</th></tr></thead><tbody><tr><td>user</td><td>Machine account ending in $ (e.g., THM-DC$)</td><td>Human user account (not ending in $)</td></tr><tr><td>Source host</td><td>Another domain controller</td><td>A workstation or non-DC server</td></tr><tr><td>Frequency</td><td>Regular intervals matching replication schedule</td><td>One-time or burst of requests</td></tr><tr><td>Scope</td><td>Specific partition changes</td><td>Requesting all credentials (-just-dc-ntlm or full dump)</td></tr></tbody></table>

Detection with splunk

```splunk
index=task5 EventCode=4662 "1131f6ad" user!="*$"
| table _time, user, Access_Mask, Properties
| sort _time
```

#### Getting the Source IP (Correlation 4662 4624)

```splunk
index=task5 EventCode=4624 Logon_ID={LOGON_ID}
| table _time, host, user, Source_Network_Address, Logon_Type
```

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

***

## Detection NTDS.dit Extraction

#### How NTDS.dit Extraction Works

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

#### Method 1: vssadmin Shadow Copy

```shellscript
C:\Windows\System32> vssadmin create shadow /for=C:
C:\Windows\System32> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\temp\ntds.dit
C:\Windows\System32> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\temp\SYSTEM
C:\Windows\System32> vssadmin delete shadows /shadow={shadow-id} /quiet
```

#### Detecting

1. search for the shadow copy creation

```splunk
index=task6 EventCode=1 Image="*\vssadmin.exe" CommandLine="*create shadow*"
| table _time, host, User, ParentImage, Image, CommandLine
```

2. look for `copy` commands that target

```splunk
index=task6 EventCode=1 CommandLine="*HarddiskVolumeShadowCopy*" (CommandLine="*ntds*" OR CommandLine="*SYSTEM*")
| table _time, host, User, ParentImage, Image, CommandLine
```

#### Method 1: ntdsutil Malicious Use

```shellscript
C:\Windows\System32> ntdsutil "ac i ntds" "ifm" "create full C:\temp" q q
```

#### Detecting

1. looking for `ntdsutil.exe` execution

```splunk
index=task6 EventCode=1 Image="*\ntdsutil.exe"
| table _time, host, User, ParentImage, Image, CommandLine
```

2. check if the extraction produced the expected file

```splunk
index=task6 EventCode=11 TargetFilename="*ntds.dit" Image="*\\ntdsutil.exe"
| table _time, Image, TargetFilename
```

#### DCSync vs NTDS.dit Extraction

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>
