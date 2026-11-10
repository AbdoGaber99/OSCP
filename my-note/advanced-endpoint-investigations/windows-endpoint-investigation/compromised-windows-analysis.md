# Compromised Windows Analysis

## <mark style="color:red;">Investigating Persistence</mark>

### Scheduled Tasks

## <mark style="color:red;">Investigating Recently Accessed Files</mark>

### LNK Files

**LNK files** store important metadata that can reveal attacker activity, even if the original file is deleted.\
They can contain:

* Full path to the target file or folder.
* Target file name and size.
* Creation, modification, and last access timestamps.
* Drive serial number of the disk containing the file.
* Volume label of the disk.
* Sometimes, network share path if the file was on a network location.

```
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent Items
```

### Parsing LNK Files

`LECmd`

```
.\LECmd.exe -d C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent --csvf Parsed-LNK.csv --csv C:\Users\Administrator\Desktop
```

open with TimeLine Explorer

## <mark style="color:red;">Investigating File Execution</mark>

### Prefetch Files

Prefetch files are part of the Windows operating system that keeps track of your file execution activities

`.pf` prefetch file contains:

* **Program name** that was executed.
* **Full path** of the program.
* **Number of times** the program has been executed.
* **Last eight execution timestamps** (date and time).
* **List of DLLs and other files** loaded when the program ran.

```
C:\Windows\Prefetch
```

### Parsing Prefetch Files

`PECmd`

```powershell
.\PECmd.exe -d "C:\Windows\Prefetch" --csv C:\Users\Administrator\Desktop --csvf Prefetch-Parsed.csv
```

analysis with TimeLine Explorer

## <mark style="color:red;">The Dig of Executable</mark>

### Amcache File

We want to parse the **Amcache.hve** file to extract forensic metadata about executed or installed programs

PATH

```
C:\Windows\appcompat\Programs\Amcache.hve
```

extract Amcache csv file

```powershell
.\AmcacheParser.exe -f "C:\Windows\appcompat\Programs\Amcache.hve" --csv C:\Users\Administrator\Desktop --csvf Amcache_Parsed.csv
```

