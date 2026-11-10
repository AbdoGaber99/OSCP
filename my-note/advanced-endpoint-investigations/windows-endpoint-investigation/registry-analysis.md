# Registry Analysis

## **Structure of the Registry:**

The registry on any Windows system contains the following five root keys:

1. <mark style="color:$warning;">HKEY\_CURRENT\_USER</mark>
2. <mark style="color:$warning;">HKEY\_USERS</mark>
3. <mark style="color:$warning;">HKEY\_LOCAL\_MACHINE</mark>
4. HKEY\_CLASSES\_ROOT
5. HKEY\_CURRENT\_CONFIG

#### hive offline

1. DEFAULT (mounted on `HKEY_USERS\DEFAULT`)
2. SAM (mounted on `HKEY_LOCAL_MACHINE\SAM`)
3. SECURITY (mounted on `HKEY_LOCAL_MACHINE\Security`)
4. SOFTWARE (mounted on `HKEY_LOCAL_MACHINE\Software`)
5. SYSTEM (mounted on `HKEY_LOCAL_MACHINE\System`)

#### Data Acquisition

1. KAPE
2. FTK Imager
3. Autopsy

#### Registry Analysis Tools

1. Registry Explorer
2. RegRipper
3. ShellBagsExplorer

## Investigate artifact&#x20;

**OS Version**

`SOFTWARE\Microsoft\Windows NT\CurrentVersion`

**Computer Name**

`SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName`&#x20;

**Time Zone Information:**

`SYSTEM\CurrentControlSet\Control\TimeZoneInformation`

**Network Interfaces and Past Networks:**

`SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces`

**Autostart Programs (Autoruns)**

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce`

`SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`

`SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer\Run`

`SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

**service**

`SYSTEM\CurrentControlSet\Services`

**SAM hive and user information:**

`SAM\Domains\Account\Users`

#### **Recent Files**

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`

#### **Office Recent Files**

`NTUSER.DAT\Software\Microsoft\Office\15.0\Word`

`NTUSER.DAT\Software\Microsoft\Office\VERSION\UserMRU\LiveID_####\FileMRU`

#### **ShellBags**

**USRCLASS.DAT**

`Local Settings\Software\Microsoft\Windows\Shell\Bags`

`Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

**NTUSER.DAT**

`Software\Microsoft\Windows\Shell\Bags`

`Software\Microsoft\Windows\Shell\BagMRU`

#### **Open/Save and LastVisited Dialog MRUs:**

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU`

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU`

#### **Windows Explorer Address/Search Bars:**

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`

`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery`

**UserAssist**

`NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\{GUID}\Count`

**ShimCache**

`SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache`

**AmCache**

`C:\Windows\appcompat\Programs\Amcache.hve`

**BAM/DAM**

`SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}`

`SYSTEM\CurrentControlSet\Services\dam\UserSettings\{SID}`

{% file src="../../../.gitbook/assets/WindowsForensicsCheatsheet-TryHackMe-1642092762578.pdf" %}

