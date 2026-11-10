---
description: Network Activity and Post-Exploitation behavior
---

# Windows Memory Network

### Learning Objectives

* Identify network connections in a memory dump.
* Identify suspicious ports and remote endpoints.
* Link connections to processes.
* Detect reverse shells and memory injections in a memory dump.
* Trace PowerShell and C2 activity in memory.

## Analyzing Active Connections

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.netscan > netscan.txt
```

```shell-session
cat netscan.txt |grep LISTENING
```

## Investigating Remote Access and C2 Communications

#### Confirming Process Relationships

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.pslist > pslist.txt
```

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.cmdline > cmdline.txt
```



#### Scanning for Code Injection: Detecting Meterpreter

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.malfind --pid 10032 > malfind_10032.txt
```

#### process dump

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.memmap --pid 10032 --dump
```

#### Confirming Meterpreter with YARA

```
vol -f THM-WIN-001_071528_07052025.mem windows.vadyarascan --pid 10032 --yara-file meterpreter.yar
```

## Post-Exploitation Communication

```
strings pid.10084.dmp | grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort -u
```

