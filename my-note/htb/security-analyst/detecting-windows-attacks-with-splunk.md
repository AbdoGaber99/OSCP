# Detecting Windows Attacks with Splunk

## Leveraging Windows Event Logs

### Domain Reconnaissance

* Common native tools/commands utilized for domain reconnaissance

```
whoami /all
wmic computersystem get domain
net user /domain
net group "Domain Admins" /domain
arp -a
nltest /domain_trusts
```

* **User/Domain Reconnaissance Using BloodHound/SharpHound**
* [list of LDAP filters frequently used by reconnaissance tools](https://techcommunity.microsoft.com/t5/microsoft-defender-for-endpoint/hunting-for-reconnaissance-activities-using-ldap-search-filters/ba-p/824726).

**Detecting User/Domain Recon With Splunk**

```splunk
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventID=1 earliest=1690447949 latest=1690450687
| search process_name IN (arp.exe,chcp.com,ipconfig.exe,net.exe,net1.exe,nltest.exe,ping.exe,systeminfo.exe,whoami.exe) OR (process_name IN (cmd.exe,powershell.exe) AND process IN (*arp*,*chcp*,*ipconfig*,*net*,*net1*,*nltest*,*ping*,*systeminfo*,*whoami*))
| stats values(process) as process, min(_time) as _time by parent_process, parent_process_id, dest, user
| where mvcount(process) > 3
```

**Detecting Recon By Targeting BloodHound**

```splunk
index=main earliest=1690195896 latest=1690285475 source="WinEventLog:SilkService-Log"
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, ProcessId, DistinguishedName, SearchFilter
| sort 0 _time
| search SearchFilter="*(samAccountType=805306368)*"
| stats min(_time) as _time, max(_time) as maxTime, count, values(SearchFilter) as SearchFilter by ComputerName, ProcessName, ProcessId
| where count > 10
| convert ctime(maxTime)
```

Q1: Modify and employ the Splunk search provided at the end of this section on all ingested data (All time) to find all process names that made LDAP queries where the filter includes the string _(samAccountType=805306368)_. Enter the missing process name from the following list as your answer. N/A, Rubeus, SharpHound, mmc, powershell, \_

```splunk
index=main  source="WinEventLog:SilkService-Log"
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, ProcessId, DistinguishedName, SearchFilter
| sort 0 _time
| search SearchFilter="*(samAccountType=805306368)*"
| stats min(_time) as _time, max(_time) as maxTime, count, values(SearchFilter) as SearchFilter by ComputerName, ProcessName, ProcessId
| convert ctime(maxTime)
| stats count by ProcessName
```

<figure><img src="../../../.gitbook/assets/02 (3).png" alt=""><figcaption></figcaption></figure>

***

### Password Spraying

**Password Spraying Detection**

* `4768 and ErrorCode 0x6 - Kerberos Invalid Users`
* `4768 and ErrorCode 0x12 - Kerberos Disabled Users`
* `4776 and ErrorCode 0xC0000064 - NTLM Invalid Users`
* `4776 and ErrorCode 0xC000006A - NTLM Wrong Password`
* `4648 - Authenticate Using Explicit Credentials`
* `4771 - Kerberos Pre-Authentication Failed`

### Detecting Password Spraying With Splunk

```splunk
index=main source="WinEventLog:Security" EventCode=4625
| bin span=15m _time
| stats values(user) as Users, dc(user) as dc_user by src, Source_Network_Address, dest, EventCode, Failure_Reason
```

<figure><img src="../../../.gitbook/assets/03 (3).png" alt=""><figcaption></figcaption></figure>

***

## Detecting Responder-like Attacks

```splunk
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=22
| search QueryResults="*10.10.0.221*"
| stats count by QueryName
```

<figure><img src="../../../.gitbook/assets/04 (3).png" alt=""><figcaption></figcaption></figure>

***

## Detecting Kerberoasting/AS-REProasting

### Kerberoasting

* Detection Logic

```
IF EventID=4769
AND NOT followed by EventID=4624 OR 4648
within short timeframe
THEN suspicious Kerberoasting activity

```

### Detecting Kerberoasting With Splunk

```splunk
index=main EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc) 
| dedup RecordNumber 
| rex field=user "(?<username>[^@]+)"
| table _time, ComputerName, EventCode, name, username, Account_Name, Account_Domain, src_ip, service_name, Ticket_Options, Ticket_Encryption_Type, Target_Server_Name, Additional_Information
```

**Detecting Kerberoasting - SPN Querying**

```splunk
index=main source="WinEventLog:SilkService-Log" 
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, DistinguishedName, SearchFilter 
| search SearchFilter="*(&(samAccountType=805306368)(servicePrincipalName=*)*"
```

**Detecting Kerberoasting - TGS Requests**

```splunk
index=main EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| bin span=2m _time 
| search username!=*$ 
| stats values(EventCode) as Events, values(service_name) as service_name, values(Additional_Information) as Additional_Information, values(Target_Server_Name) as Target_Server_Name by _time, username
| where !match(Events,"4648")
```

**Detecting Kerberoasting Using Transactions - TGS Requests**

```splunk
index=main EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| search username!=*$ 
| transaction username keepevicted=true maxspan=5s endswith=(EventCode=4648) startswith=(EventCode=4769) 
| where closed_txn=0 AND EventCode = 4769
| table _time, EventCode, service_name, username
```

### AS-REPRoasting

* Detection Logic

```
EventID=4768
AND Pre_Authentication_Type=0

```

**Detecting AS-REPRoasting - Querying Accounts With Pre-Auth Disabled** (LDAP Enumeration)

```splunk
index=main source="WinEventLog:SilkService-Log" 
| spath input=Message 
| rename XmlEventData.* as * 
| table _time, ComputerName, ProcessName, DistinguishedName, SearchFilter 
| search SearchFilter="*(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304)*"
```

**Detecting AS-REPRoasting - TGT Requests For Accounts With Pre-Auth Disabled**

```splunk
index=main earliest=1690392745 latest=1690393283 source="WinEventLog:Security" EventCode=4768 Pre_Authentication_Type=0
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip>[0-9\.]+)"
| table _time, src_ip, user, Pre_Authentication_Type, Ticket_Options, Ticket_Encryption_Type
```

<figure><img src="../../../.gitbook/assets/05 (3).png" alt=""><figcaption></figcaption></figure>

***

## Detecting Pass-the-Hash

**Windows Access Token**

* User Name
* Groups
* Privilege
* Logon session ID
  * Username
  * Domain
  * NTLM hash / Kerberos

runas without netonly

```
Event 4624
LogonType = 2 (Interactive)
```

runas with netonly

```
Event 4624
LogonType = 9 (NewCredentials)
```

### Detecting Pass-the-Hash With Splunk

```splunk
index=main source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo
| table _time, ComputerName, EventCode, user, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process
```

```splunk
index=main earliest=1690450689 latest=1690451116 (source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=10 TargetImage="C:\\Windows\\system32\\lsass.exe" SourceImage!="C:\\ProgramData\\Microsoft\\Windows Defender\\platform\\*\\MsMpEng.exe") OR (source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo)
| sort _time, RecordNumber
| transaction host maxspan=1m endswith=(EventCode=4624) startswith=(EventCode=10)
| stats count by _time, Computer, SourceImage, SourceProcessId, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process
| fields - count
```

<figure><img src="../../../.gitbook/assets/06 (2).png" alt=""><figcaption></figcaption></figure>

***

## Detecting Pass-the-Ticket

Splunk detection: Look for (TGS Without TGT)

* `Event ID 4769 (Kerberos Service Ticket Request)` OR&#x20;
* `Event ID 4770 (Kerberos Service Ticket was renewed)`  without a prior&#x20;
* `Event ID 4768 (Kerberos TGT Request)`

### Detecting Pass-the-Ticket With Splunk

```splunk
index=main earliest=1690392405 latest=1690451745 source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770) 
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

***

## Detecting Overpass-the-Hash

detection&#x20;

* Network(port 88 kerberos) + process name (mimikatz, rubeus)
* corelation (sysmon Event 3) with (kerberos port 88) with (process name Rubeus)

```splunk
index=main earliest=1690443407 latest=1690443544 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode=3 dest_port=88 Image!=*lsass.exe) OR EventCode=1
| eventstats values(process) as process by process_id
| where EventCode=3
| stats count by _time, Computer, dest_ip, dest_port, Image, process
| fields - count
```

***

## Detecting Golden Tickets/Silver Tickets

### Golden Ticket

### Detecting Golden Tickets With Splunk (Yet Another Ticket To Be Passed Approach)

monitor common methods of extracting the `KRBTGT` hash:

* `DCSync attack`&#x20;
* `NTDS.dit file access`
* &#x20;`LSASS memory read on the domain controller (Sysmon Event ID 10)`

```splunk
index=main source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770) 
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

### Silver Ticket

**Detecting Silver Tickets With Splunk Through User Correlation**

* Let's first create a list of users (`users.csv`) leveraging `Event ID 4720 (A user account was created)`

```splunk
index=main latest=1690448444 EventCode=4720
| stats min(_time) as _time, values(EventCode) as EventCode by user
| outputlookup users.csv
```

**Note**: `users.csv` can be downloaded from the `Resources` section of this module (upper right corner) and uploaded to Splunk by clicking `Settings` -> `Lookups` -> `Lookup table files` -> `New Lookup Table File`

````splunk
index=main latest=1690545656 EventCode=4624
| stats min(_time) as firstTime, values(ComputerName) as ComputerName, values(EventCode) as EventCode by user
| eval last24h = 1690451977
| where firstTime > last24h
```| eval last24h=relative_time(now(),"-24h@h")```
| convert ctime(firstTime)
| convert ctime(last24h)
| lookup users.csv user as user OUTPUT EventCode as Events
| where isnull(Events)
````

**Detecting Silver Tickets With Splunk By Targeting Special Privileges Assigned To New Logon**

````splunk
index=main latest=1690545656 EventCode=4672
| stats min(_time) as firstTime, values(ComputerName) as ComputerName by Account_Name
| eval last24h = 1690451977 
```| eval last24h=relative_time(now(),"-24h@h") ```
| where firstTime > last24h 
| table firstTime, ComputerName, Account_Name 
| convert ctime(firstTime)
````

## Detecting Unconstrained Delegation

* monitoring PowerShell script block logging (`Event ID 4104`) and
  * &#x20;LDAP request logging.
  * TrustedForDelegation

```splunk
index=main source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 Message="*TrustedForDelegation*" OR Message="*userAccountControl:1.2.840.113556.1.4.803:=524288*" 
| table _time, ComputerName, EventCode, Message
```

## Detecting Constrained Delegation&#x20;

* monitoring PowerShell script block logging (`Event ID 4104`) and
  * msDS-AllowedToDelegateTo
  * TrustedToAuthForDelegation

```splunk
index=main source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 Message="*msDS-AllowedToDelegateTo*" 
| table _time, ComputerName, EventCode, Message
```

**Detecting Constrained Delegation Attacks - Leveraging Sysmon Logs**

```splunk
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" 
| eventstats values(process) as process by process_id
| where EventCode=3 AND dest_port=88
| table _time, Computer, dest_ip, dest_port, Image, process
```

***

## Detecting DCSync/DCShadow

#### DCSync Attack Detection (Key Indicator)

* Monitor **Event ID 4662** (Directory Service access)
* Look for the GUID:

```
{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}    ##This represents DS-Replication-Get-Changes
```

* The request comes from:
  * A system that is **not a Domain Controller**
  * A user account **not normally used for replication**

### Detecting DCSync With Splunk

```splunk
index=main EventCode=4662 Message="*Replicating Directory Changes*"
| rex field=Message "(?P<property>Replicating Directory Changes.*)"
| table _time, user, object_file_name, Object_Server, property
```

### DCShadow

* `Event ID 4742 (Computer account was changed)` logs changes related to computer objects, including `ServicePrincipalName`

### Detecting DCShadow With Splunk

```splunk
index=main EventCode=4742 
| rex field=Message "(?P<gcspn>GC\/[a-zA-Z0-9\.\-\/]+)" 
| table _time, ComputerName, Security_ID, Account_Name, user, gcspn 
| search gcspn=*
```

***

## Leveraging Zeek Logs

### Detecting RDP Brute Force Attacks

**How RDP Traffic Looks Like**

* Authentication
* Certificate exchange
* Close RDP Connection Alert(21)
* RDP Username

**Detecting RDP Brute Force**

* Multiple failed RDP attempts
* Same src\_ip → many sessions
* Short time window

### Detecting RDP Brute Force Attacks With Splunk & Zeek Logs

```splunk
index="rdp_bruteforce" sourcetype="bro:rdp:json"
| bin _time span=5m
| stats count values(cookie) by _time, id.orig_h, id.resp_h
| where count>30
```

<figure><img src="../../../.gitbook/assets/07 (7).png" alt=""><figcaption></figcaption></figure>

***

### Detecting Beaconing Malware

#### Malware Beaconing Behavior

* **Periodic communication**: Infected systems regularly contact their C2 servers.
* **Small data packets**: Typically minimal data is sent each time.
* **Intervals**:
  * Fixed (exact timing)
  * Jittered (slightly varied)
  * Complex/smart schedules (based on malware logic)
* **Protocols used**: HTTP/HTTPS, DNS, ICMP (ping)
* **Targets**: Usually a single IP or domain controlled by the attacker
* **Purpose**: Maintain connection, receive commands, exfiltrate data

### Detecting Beaconing Malware With Splunk & Zeek Logs

```splunk
index="cobaltstrike_beacon" sourcetype="bro:http:json" 
| sort 0 _time
| streamstats current=f last(_time) as prevtime by src, dest, dest_port
| eval timedelta = _time - prevtime
| eventstats avg(timedelta) as avg, count as total by src, dest, dest_port
| eval upper=avg*1.1
| eval lower=avg*0.9
| where timedelta > lower AND timedelta < upper
| stats count, values(avg) as TimeInterval by src, dest, dest_port, total
| eval prcnt = (count/total)*100
| where prcnt > 90 AND total > 10
```

***

### Detecting Nmap Port Scanning

Nmap Port Scanning – Key Behaviors

* Same source IP connects to **many different ports** on a target
* Attempts happen within a **short time window**
* Connections usually contain **no real payload** (just TCP handshake)
* Open ports may return **service banners** (service/version info)
* High volume of connection attempts compared to normal traffic

### Detecting Nmap Port Scanning With Splunk & Zeek Logs

```splunk
index="cobaltstrike_beacon" sourcetype="bro:conn:json" orig_bytes=0 dest_ip IN (192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8) 
| bin span=5m _time 
| stats dc(dest_port) as num_dest_port by _time, src_ip, dest_ip 
| where num_dest_port >= 3
```

<figure><img src="../../../.gitbook/assets/08 (8).png" alt=""><figcaption></figcaption></figure>

***

### Detecting Kerberos Brute Force Attacks

* User Exist but need pre-authentication

```
KRB5KDC_ERR_PREAUTH_REQUIRED
```

* User not found

```
KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN
```

Network Indicators

* High volume of **AS-REQ** requests to the KDC
* Many responses containing:
  * **KRB5KDC\_ERR\_C\_PRINCIPAL\_UNKNOWN** (invalid username)
* Requests usually originate from the **same source IP**

### Detecting Kerberos Brute Force Attacks With Splunk & Zeek Logs

```splunk
index="kerberos_bruteforce" sourcetype="bro:kerberos:json"
error_msg!=KDC_ERR_PREAUTH_REQUIRED
success="false" request_type=AS
| bin _time span=5m
| stats count dc(client) as "Unique users" values(error_msg) as "Error messages" by _time, id.orig_h, id.resp_h
| where count>30
```

***

### Detecting Kerberoasting

### Detecting Kerberoasting With Splunk & Zeek Logs

```splunk
index="sharphound" sourcetype="bro:kerberos:json"
request_type=TGS cipher="rc4-hmac" 
forwardable="true" renewable="true"
| table _time, id.orig_h, id.resp_h, request_type, cipher, forwardable, renewable, client, service
```

***

## Detecting Golden Tickets

#### Detecting Golden Tickets With Splunk & Zeek Logs

```splunk
index="golden_ticket_attack" sourcetype="bro:kerberos:json"
| where client!="-"
| bin _time span=1m 
| stats values(client), values(request_type) as request_types, dc(request_type) as unique_request_types by _time, id.orig_h, id.resp_h
| where request_types=="TGS" AND unique_request_types==1
```

***

## Detecting Cobalt Strike's PSExec

**Indicators of PSExec Abuse**

* Unusual SMB communication between hosts
* Access to **ADMIN$** from non-admin systems
* Sudden service creation events
* Workstations performing admin-like behavior

#### Detecting Cobalt Strike's PSExec With Splunk & Zeek Logs

```splunk
index="cobalt_strike_psexec"
sourcetype="bro:smb_files:json"
action="SMB::FILE_OPEN" 
name IN ("*.exe", "*.dll", "*.bat")
path IN ("*\\c$", "*\\ADMIN$")
size>0
```

***

### Detecting Exfiltration (HTTP)

#### Detecting HTTP Exfiltration With Splunk & Zeek Logs

```splunk
index="cobaltstrike_exfiltration_http" sourcetype="bro:http:json" method=POST
| stats sum(request_body_len) as TotalBytes by src, dest, dest_port
| eval TotalBytes = TotalBytes/1024/1024
```

***

### Detecting Exfiltration (DNS)

#### Detection Indicators

* Unusually long DNS query names
* Random or encoded-looking subdomains
* High volume of DNS requests from a single host
* Repeated queries to unknown or rare domains
* High entropy (looks like gibberish, not normal words)

#### Detecting DNS Exfiltration With Splunk & Zeek Logs

```splunk
index=dns_exf sourcetype="bro:dns:json"
| eval len_query=len(query)
| search len_query>=40 AND query!="*.ip6.arpa*" AND query!="*amazonaws.com*" AND query!="*._googlecast.*" AND query!="_ldap.*"
| bin _time span=24h
| stats count(query) as req_by_day by _time, id.orig_h, id.resp_h
| where req_by_day>60
| table _time, id.orig_h, id.resp_h, req_by_day
```

### Detecting Ransomware

1. `File Overwrite Approach`:
2. `File Renaming Approach`

#### Detecting Ransomware With Splunk & Zeek Logs (Excessive Overwriting)

```splunk
index="ransomware_open_rename_sodinokibi" sourcetype="bro:smb_files:json" 
| where action IN ("SMB::FILE_OPEN", "SMB::FILE_DELETE")
| bin _time span=5m
| stats count by _time, source, action
| where count>30 
| stats sum(count) as count values(action) dc(action) as uniq_actions by _time, source
| where uniq_actions==2 AND count>100 
```

#### Detecting Ransomware With Splunk & Zeek Logs (Excessive Renaming With The Same Extension)

```splunk
index="ransomware_new_file_extension_ctbl_ocker" sourcetype="bro:smb_files:json" action="SMB::FILE_RENAME" 
| bin _time span=5m
| rex field="name" "\.(?<new_file_name_extension>[^\.]*$)"
| rex field="prev_name" "\.(?<old_file_name_extension>[^\.]*$)"
| stats count by _time, id.orig_h, id.resp_p, name, source, old_file_name_extension, new_file_name_extension,
| where new_file_name_extension!=old_file_name_extension
| stats count by _time, id.orig_h, id.resp_p, source, new_file_name_extension
| where count>20
| sort -count
```
