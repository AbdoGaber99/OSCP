# Set up a sysmon

## Step 1: Download Sysmon

Download Sysmon from Microsoft Sysinternals:

https://learn.microsoft.com/sysinternals/downloads/sysmon

Or use PowerShell:

```
Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile Sysmon.zip
Expand-Archive Sysmon.zip
cd Sysmon
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 205831.png" alt=""><figcaption></figcaption></figure>

***

## Step 2: Download a Sysmon Configuration File

Sysmon requires a configuration file to be useful.

We will use a well-known community config (SwiftOnSecurity):

```
Invoke-WebRequest -Uri https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile sysmonconfig.xml
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 205900.png" alt=""><figcaption></figcaption></figure>

***

## Step 3: Install Sysmon

Run Sysmon with the configuration file:

```
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```

✔ This installs Sysmon and starts monitoring the system immediately.

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 205943.png" alt=""><figcaption></figcaption></figure>

***

## Step 4: Verify Sysmon Installation

Check if the service is running:

```
Get-Service Sysmon64
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 210011.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 210105.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-17 210203.png" alt=""><figcaption></figcaption></figure>
