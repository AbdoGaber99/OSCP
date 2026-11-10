# Analyzing Evil With Sysmon & Event Logs

### Detection Example 1: Detecting DLL Hijacking

In the case of detecting DLL hijacks, we change the "include" to "exclude"

```xml
<RuleGroup name="ImageLoad Monitoring" groupRelation="or">
	<ImageLoad onmatch="exclude">
	</ImageLoad>
</RuleGroup>
```

updated Sysmon configuration, execute the following.

```shellscript
sysmon.exe -c sysmonconfig-export.xml
```

<figure><img src="../../../.gitbook/assets/Screenshot 2025-12-22 111121.png" alt=""><figcaption></figcaption></figure>

which involve renaming `reflective_dll.x64.dll` to `WININET.dll`, moving `calc.exe` from `C:\Windows\System32` along with `WININET.dll` to a writable directory (such as the `Desktop` folder), and executing `calc.exe`, we achieve success

<figure><img src="../../../.gitbook/assets/Screenshot 2025-12-22 111246.png" alt=""><figcaption></figcaption></figure>

ext, we analyze the impact of the hijack. First, we filter the event logs to focus on `Event ID 7`&#x20;

<figure><img src="../../../.gitbook/assets/Screenshot 2025-12-22 111448.png" alt=""><figcaption></figcaption></figure>

we search for instances of "calc.exe", by clicking "Find..." to identify the DLL load associated with our hijack.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-12-22 111611.png" alt=""><figcaption></figcaption></figure>

***

### Detection Example 2: Detecting Unmanaged PowerShell/C-Sharp Injection

* loaded `clr.dll`  `clrjit.dll`
