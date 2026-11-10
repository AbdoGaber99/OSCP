# Monitoring Active Directory

### Protocol Summary

<table><thead><tr><th width="123">Protocol</th><th width="227">Ports</th><th>What It Does</th><th>Normal Usage</th></tr></thead><tbody><tr><td>Kerberos</td><td>88</td><td>Default authentication in</td><td>User logins, service access, ticket requests</td></tr><tr><td>LDAP</td><td>389, 636, 3268, 3269</td><td>Directory queries and modifications</td><td>User lookups, group membership checks, and address book queries</td></tr><tr><td>SMB</td><td>445 (139 for legacy sessions)</td><td>File sharing, remote administration</td><td>Accessing shared folders, printers, and administrative tools</td></tr><tr><td>RDP</td><td>3389</td><td>Interactive remote desktop</td><td>Help desk support, server administration</td></tr><tr><td>Name Resolution (Legacy)</td><td>137, 138, 5355</td><td>(/ - 137/138) and (5355)</td><td>Fallback when fails, for older applications</td></tr></tbody></table>

### Kerberos Authentication

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770157962524.svg" alt="Diagram showing the three-step Kerberos authentication flow: TGT request to DC, TGS request for a service, and session creation on the target" width="100%">

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770130895293.svg" alt="Kerberos events as they appear in logs" width="100%">

### Encryption Types in 4768/4769

* **RC4** encryption appears in environments with older systems or applications that don't support AES.
* And **AES-256** encryption, which modern systems use.

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770130895307.png" alt="Event 4768 details showing the Ticket Encryption Type field with AES-256 and RC4 values" width="375">

* filter for TGT requests in Splunk

```splunk
index=* EventCode=4768
| table _time, Account_Name, Client_Address, Ticket_Encryption_Type
```

### NTLM Authentication

<table><thead><tr><th width="80">Step</th><th width="273">What Happens</th><th width="94">Event ID</th><th>Where Logged</th></tr></thead><tbody><tr><td>1</td><td>The target server asks the DC to validate credentials</td><td>4776</td><td>Domain Controller</td></tr><tr><td>2</td><td>Session created on target</td><td>4624</td><td>Target server</td></tr></tbody></table>

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770233261725.svg" alt="Diagram showing the two-step NTLM authentication flow: target server forwards credentials to DC for validation, then session is created on the target" width="100%">

* NTLM authentication attempts on the in Splunk

```
index=* EventCode=4776
| table _time, Logon_Account, Source_Workstation
```

***

### Account Lifecycle Events

<table><thead><tr><th width="114">Event ID</th><th width="273">What Happened</th></tr></thead><tbody><tr><td>4720</td><td>Account created</td></tr><tr><td>4722</td><td>Account enabled</td></tr><tr><td>4724</td><td>Password reset attempted</td></tr><tr><td>4725</td><td>Account disabled</td></tr><tr><td>4740</td><td>Account locked out</td></tr></tbody></table>

### Group Membership Events

<table><thead><tr><th width="114">Event ID</th><th width="206">What Happened</th><th width="263">Group Scope</th></tr></thead><tbody><tr><td>4728</td><td>Member added to the global security group</td><td>Domain-wide</td></tr><tr><td>4732</td><td>Member added to local security group</td><td>Machine-level (domain local on DCs)</td></tr><tr><td>4756</td><td>Member added to universal security group</td><td>Entire forest</td></tr></tbody></table>

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770237504840.png" alt="Event viewer showing group membership change details" width="100%">

* see group membership changes for any of these groups

```powershell
index=* (EventCode=4728 OR EventCode=4732 OR EventCode=4756)
| table _time, Member_Account_Name, Group_Name, Subject_Account_Name
```

### Directory Service Events

Event 5136 logs attribute-level modifications to Active Directory objects

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770302179225.png" alt="Event 5136 showing logon script configuration change" width="65%">

### Common Service Name Patterns

| ServiceName Pattern  | What It Represents     |
| -------------------- | ---------------------- |
| krbtgt               | TGT renewal requests   |
| cifs/THM-SHR-SRV     | File share access      |
| ldap/THM-DC          | Directory queries      |
| http/THM-WEB-SRV     | Web application access |
| MSSQLSvc/THM-SQL-SRV | SQL Server access      |
| HOST/THM-IT-DESK     | General host services  |

***

### Advanced Audit Policy Configuration

<img src="https://cdn-images.tryhackme.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1770149572975.png" alt="Group Policy Management Editor showing Advanced Audit Policy Configuration" width="100%">

| Category           | Subcategory               | Setting          | Events Generated       |
| ------------------ | ------------------------- | ---------------- | ---------------------- |
| Account Logon      | Credential Validation     | Success, Failure | 4776 ()                |
| Account Logon      | Authentication Service    | Success, Failure | 4768, 4771             |
| Account Logon      | Service Ticket Operations | Success, Failure | 4769                   |
| Account Management | User Account Management   | Success, Failure | 4720, 4722, 4724, 4725 |
| Account Management | Security Group Management | Success, Failure | 4728, 4732, 4756       |
| DS Access          | Directory Service Changes | Success, Failure | 5136                   |
| Logon/Logoff       | Logon                     | Success, Failure | 4624, 4625             |
| Object Access      | File Share                | Success          | 5140                   |

### Verifying Your Configuration

```
# View all audit policy settings
auditpol /get /category:*
```
