# Active Directory Introduction and Enumeration

## Introduction

#### 🏛️ **Default Domain Groups**

| **Domain Admins**      | Full control over the entire domain; members are automatically local admins on all domain machines. |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| **Domain Users**       | All regular users in the domain; every new user is added here by default.                           |
| **Domain Computers**   | Contains all computers that join the domain.                                                        |
| **Domain Controllers** | Contains all Domain Controller servers in the domain.                                               |
| **Administrators**     | Full administrative rights on Domain Controllers and domain resources.                              |
| **Users**              | Standard users with limited privileges.                                                             |
| **Guests**             | Temporary users with minimal privileges.                                                            |
| **Account Operators**  | Can create and manage user and group accounts (except admin accounts).                              |
| **Server Operators**   | Can manage and restart servers but not domain-wide settings.                                        |
| **Backup Operators**   | Can back up and restore files without full admin rights.                                            |
| **Print Operators**    | Can manage printers in the domain.                                                                  |
| **Replicator**         | Used for file replication between Domain Controllers.                                               |

#### 🌐 **Forest-Level Groups (Created in the Root Domain)**

| **Group**                       | **Description**                                              |
| ------------------------------- | ------------------------------------------------------------ |
| **Enterprise Admins**           | Full administrative rights across all domains in the forest. |
| **Schema Admins**               | Can modify the AD Schema (structure of objects).             |
| **Group Policy Creator Owners** | Can create and manage Group Policies.                        |

### Active Directory - Manual Enumeration

* Enumerate Active Directory using legacy Windows applications
* Use PowerShell and .NET to perform additional AD enumeration

#### Enumeration Using Legacy Windows Tools

* **CN (Common Name)**
* **DC (Domain Component)**
* **DN (Distinguished Name)**\
  `CN=Abdo.Gaber,CN=Users,DC=corp,DC=com`

```powershell
net user /domain
```

```powershell
net user "abdo.gaber" /domain
```

```powershell
net group /domain
```

```powershell
net group "Domain Admins" /domain
```

#### Enumerating Active Directory using PowerShell and .NET

1. create script

<pre class="language-powershell" data-title="function.ps1"><code class="lang-powershell">function LDAPSearch {
<strong><a data-footnote-ref href="#user-content-fn-1">    param (</a>
</strong>        [string]$LDAPQuery
    )
    $PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
    $DistinguishedName = ([adsi]'').distinguishedName
    $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")
    $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)
    return $DirectorySearcher.FindAll()
}
</code></pre>

2. Import module

```powershell
Import-Module .\function.ps1
```

3. let's query

```powershell
LDAPSearch -LDAPQuery "(samAccountType=805306368)"
```

```powershell
LDAPSearch -LDAPQuery "(objectclass=group)"
```

* enumerate every group available in the domain and also display the user members,

```powershell
foreach ($group in $(LDAPSearch -LDAPQuery "(objectCategory=group)")) { $group.properties | select {$_.cn}, {$_.member}}
```

### AD Enumeration with PowerView

#### Enumerating Operating Systems

1. Import module

```powershell
Import-Module .\PowerView.ps1
```

2. information about the domain

```powershell
Get-NetDomain
```

3. list of all users in the domain&#x20;

```powershell
Get-NetUser | select cn,pwdlastset,lastlogon
```

4. enumerate groups

```powershell
Get-NetGroup | select cn
Get-NetGroup "Sales Department" | select member
```

5. Enumerating Operating Systems

```powershell
Get-NetComputer | select operatingsystem,dnshostname
```

#### Enumerating  Permissions and Logged on Users

6. current user has administrative permissions on any computers

```powershell
Find-LocalAdminAccess
```

7. logged in users

```powershell
Get-NetSession -ComputerName filesrv01 -Verbose
```

```
.\psloggedon \\filesrv01
```

#### Enumeration Through Service Principal Names

* **Service Principal Name (SPN):** The SPN links a service name to a user account or service account in the domain (\<ServiceName>\\\<HostName>:\<Port>)

```powershell
Get-NetUser -SPN | select samaccountname,serviceprincipalname
```

```powershell
setspn -L iis_service
```



[^1]: 
