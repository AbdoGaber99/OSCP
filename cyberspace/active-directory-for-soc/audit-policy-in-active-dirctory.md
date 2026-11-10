# Audit Policy in Active dirctory

1. Account Logon

```
Audit Kerberos Authentication Service: Success, Failure
Audit Kerberos Service Ticket Operations: Success, Failure
Audit Credential Validation: Success, Failure
```

2. Logon / Logoff

```
Audit Logon: Success, Failure
Audit Logoff: Success
Audit Special Logon: Success
```

3. Account Management

```
Audit User Account Management: Success, Failure
Audit Computer Account Management: Success, Failure
Audit Security Group Management: Success, Failure
```

4. Directory Service Access

```
Audit Directory Service Access: Success, Failure
Audit Directory Service Changes: Success, Failure
```

5. Privilege Use

```
Audit Sensitive Privilege Use: Success, Failure
```

6. Enable Command Line logging

```
Computer Configuration → Administrative Templates → System → Audit Process Creation
→ Include command line in process creation events = Enabled
```

***

Active Directory Audit Policy Setup (Domain Controller)

Step-by-Step Configuration

### 1. Open Group Policy Management

Run:

```
gpmc.msc
```

Navigate to:

* Forest → Domains → Your Domain → Domain Controllers

***

### 2. Edit Domain Controller GPO

* Open **Default Domain Controllers Policy** (or create a new GPO)
* Click **Edit**

***

### 3. Enable Advanced Audit Policy

Go to:

```
Computer Configuration 
→ Policies 
→ Windows Settings 
→ Security Settings 
→ Advanced Audit Policy Configuration 
→ Audit Policies
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 214431.png" alt=""><figcaption></figcaption></figure>

***

### 4. Enable Key Audit Categories

#### 🔐 Account Logon

* Kerberos Authentication Service (Success, Failure)
* Kerberos Service Ticket Operations (Success, Failure)
* Credential Validation (Success, Failure)

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 214616.png" alt=""><figcaption></figcaption></figure>

#### 👤 Logon/Logoff

* Logon (Success, Failure)
* Logoff (Success)
* Special Logon (Success)

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 215053.png" alt=""><figcaption></figcaption></figure>

#### 🧑‍💼 Account Management

* User Account Management (Success, Failure)
* Group Management (Success, Failure)
* Computer Account Management (Success, Failure)

#### 📁 Directory Service Access (Critical for AD monitoring)

* Directory Service Access (Success, Failure)
* Directory Service Changes (Success, Failure)

#### ⚙️ Policy Change

* Audit Policy Change (Success, Failure)
* Authentication Policy Change (Success, Failure)

#### ⚙️ Process Creation

* Audit Process Creation (Success)

***

### 5. Enable Command-Line Logging

Go to:

```
Computer Configuration → Administrative Templates → System → Audit Process Creation
```

Enable:

* “Include command line in process creation events”

***

### 6. Apply Policy

Run:

```
gpupdate /force
```





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

```
# View all audit policy settings
auditpol /get /category:*
```
