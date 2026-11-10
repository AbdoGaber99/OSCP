# Event Tracing for Windows (ETW)



#### ETW Providers

```
logman.exe query providers
```

* kernel provider

```powershell
Microsoft-Windows-Kernel-Process
Microsoft-Windows-Kernel-Thread
Microsoft-Windows-Kernel-Network
Microsoft-Windows-Kernel-File
Microsoft-Windows-Kernel-Registry
Microsoft-Windows-Kernel-Image
```

* PowerShell & Scripting

```
Microsoft-Windows-PowerShell
Microsoft-Windows-PowerShell-Operational
Microsoft-Windows-DotNETRuntime
```

* Network

```
Microsoft-Windows-TCPIP
Microsoft-Windows-WinINet
Microsoft-Windows-WinHttp
```

#### use ETW in realtime&#x20;

```ps
logman start HuntTrace `
-p Microsoft-Windows-Kernel-Process `
-o C:\Temp\hunt.etl -ets

```

```
logman query -ets
```

```
logman stop HuntTrace -ets
```

<figure><img src="../../../.gitbook/assets/11 (13).png" alt=""><figcaption></figcaption></figure>

```
SilkETW.exe -t user -pn Microsoft-Windows-Kernel-Process -ot file -p C:\windows\temp\etw.json
```







```
SilkETW.exe -t user -pn Microsoft-Windows-Kernel-Process -ot file -p C:\windows\temp\etw.json
```

### Detection Example 2: Detecting Malicious .NET Assembly Loading

to identify malicious .NET assembly loading. This is achieved by scrutinizing the activity related to the loading of [.NET-associated DLLs](https://redhead0ntherun.medium.com/detecting-net-c-injection-execute-assembly-1894dbb04ff7), specifically `clr.dll` and `mscoree.dll`.

```
SilkETW.exe -t user -pn Microsoft-Windows-DotNETRuntime -uk 0x2038 -ot file -p C:\windows\temp\etw.json
```

* indicated by `0x2038`&#x20;
  * `JitKeyword`&#x20;
  * `InteropKeyword`&#x20;
  * `LoaderKeyword`&#x20;
  * `NGenKeyword`
