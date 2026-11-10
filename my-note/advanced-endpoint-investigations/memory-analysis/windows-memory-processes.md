# Windows Memory  Processes

## Windows Process Structures Overview

* **EPROCESS (Executive Process Block)**
  * Kernel structure that represents a process.
  * Contains PID, process name, handle table, and memory space info.
* **ETHREAD (Executive Thread Block)**
  * Kernel structure that represents a thread.
  * Contains TID, CPU context, scheduling info, and stack pointers.
* **PEB (Process Environment Block)**
  * User-space structure for process-wide data.
  * Stores loaded DLLs, command-line arguments, environment variables, and heap info.
* **TEB (Thread Environment Block)**
  * User-space structure for thread-specific data.
  * Stores Thread Local Storage (TLS), exception handling info, and stack details

```css
[ EPROCESS ]
    ├── ETHREAD #1 → [ TEB #1 ]
    ├── ETHREAD #2 → [ TEB #2 ]
    └── ETHREAD #3 → [ TEB #3 ]
    
[ PEB ]  ← مرتبط بالـ EPROCESS
```

***

### Summary

* **EPROCESS** = Process representation (Kernel).
* **ETHREAD** = Thread representation (Kernel).
* **PEB** = Process configuration (User space).
* **TEB** = Thread configuration (User space).

#### **How Do These Structures Relate?**

1. The kernel creates an EPROCESS and ETHREAD object and links them together. EPROCESS has a field called `ThreadListHead` that keeps track of all its threads
2. The virtual memory is initialized:
   * The process is assigned a virtual address space (in user mode)
   * The image (EXE) is mapped into memory
   * Space is allocated for the PEB and TEB
   * Process parameters are created
3. The PEB and TEB are initialized.
   * EPROCESS includes a pointer to the PEB
   * ETHREAD includes a pointer to the TEB
   * TEB includes a redundant pointer to the PEB
4. The thread context is set but remains suspended. This thread context includes:
   * Start address of the program
   * Stack pointer
   * Instruction pointer
5. The process is created but is still suspended
6. The primary thread of the process is resumed. This is the actual start of the process

<figure><img src="../../../.gitbook/assets/01 (17).png" alt="" width="375"><figcaption></figcaption></figure>

### What Information Can You Extract From These Structures?

#### EPROCESS STRUCTURE

```c
           
struct _EPROCESS { 
    HANDLE UniqueProcessId; // PID (Process ID) 
    LIST_ENTRY ActiveProcessLinks; // Link in active process list (Used to keep track of all active processes) 
    UCHAR ImageFileName[15]; // Short process name LARGE_INTEGER CreateTime; // Process creation time 
    LARGE_INTEGER ExitTime; // Exit time if terminated 
    PPEB Peb; // Pointer to user-mode PEB 
    HANDLE InheritedFromUniqueProcessId; // Parent PID 
    LIST_ENTRY ThreadListHead; // List of ETHREADs 
    PHANDLE_TABLE ObjectTable; // Handle table (points to opened files) 
    PVOID SectionObject; // Executable image mapping 
    PVOID VadRoot; // VAD tree for memory mapping PACCESS_TOKEN Token; // Security information 
}
```

`volatility` plugins extract information from EPROCESS

* pslist
* pstree
* psscan
* malfind
* getsids
* handles
* dlllist
* cmdline
* envars
* ldrmodules

#### THREAD STRUCTURE

```c
           
struct _ETHREAD { 
    CLIENT_ID Cid; // Thread and Process IDs 
    LARGE_INTEGER CreateTime; // Thread creation time 
    LARGE_INTEGER ExitTime; // Thread exit time 
    PVOID StartAddress; // Kernel-level entry point 
    PVOID Win32StartAddress; // User-mode entry point 
    LIST_ENTRY ThreadListEntry; // Link in EPROCESS's thread list 
    PTEB Teb; // Pointer to TEB 
    ULONG ThreadState; // Thread execution state 
    ULONG WaitReason; // Reason for being blocked 
}
```

`volatility` plugins extract information from ETHREAD:&#x20;

* threads
* ldrmodules
* apihooks
* malfind

#### PEB STRUCTURE         &#x20;

<pre class="language-c"><code class="lang-c"><strong>struct _PEB { 
</strong>    BOOLEAN BeingDebugged; // Debug flag
    PVOID ImageBaseAddress; // Base address of executable 
    PPEB_LDR_DATA Ldr; // Loader data (DLLs) 
    PRTL_USER_PROCESS_PARAMETERS ProcessParameters;// Command-line, environment variables
    ULONG NtGlobalFlag; // Debugging heap flags 
    PVOID ProcessHeap; // Default process heap 
}
</code></pre>

`volatility` plugins extract information from the Process Environment Block:

* cmdline
* envars
* ldrmodules
* malfind

#### TEB STRUCTURE         &#x20;

```c
struct _TEB { 
    PVOID EnvironmentPointer; // Pointer to env block 
    CLIENT_ID ClientId; // Thread + Process IDs 
    PVOID ThreadLocalStoragePointer; // TLS base 
    PPEB ProcessEnvironmentBlock; // Pointer to PEB 
    ULONG LastErrorValue; // Last error value 
    PVOID StackBase; // Upper bound of thread stack 
    PVOID StackLimit; // Lower bound of thread stack 
    PVOID Win32ThreadInfo; // GUI subsystem data 
}
```

`volatility` plugins extract information from the Thread Environment Block

* threads
* malfind

***

## <mark style="color:red;">Initial Triage of a New Memory Dump</mark>

#### Verifying the Memory Dump

```bash
diff acquisitionhash.txt newhash.txt
```

#### Extract Processes

* `windows.pslist`: extract a list of the **active processes at the time of the memory dump**
* `windows.psscan`: extract **all process objects,** including terminated and unlinked processes&#x20;
* `windows.pstree`: extracts a list of the **active processes at the time of the memory dump** and visualizes the parent-child relationships between them
* `windows.psxview`: uses multiple process detection techniques to gather information about processes, threads, process structures, and handles, then cross-reference and compare this information.

```
vol3 -f THM-WIN-001_071528_07052025.mem windows.pslist > pslist.txt
```

### Finding a Needle in a Haystack

1. **Suspicious Name**
2. **Suspicious Path**
3. **Masquerading**
4. **Comparing With a Baseline**

#### <mark style="color:purple;">**Comparing With a Baseline**</mark>

* compare Baseline process with current process

```bash
awk 'NR >3{print $2}' baseline/baseline.txt | sort | uniq > baseline_procs.txt
awk 'NR >3{print $3}' pslist.txt | sort | uniq > current_procs.txt
comm -13 baseline_procs.txt current_procs.txt
```

### Analysis Notes results

| pdfupdater.exe     | 3392  | 2025-05-07 07:13:05.000000 |
| ------------------ | ----- | -------------------------- |
| windows-update.exe | 10084 | 2025-05-07 07:13:05.000000 |
| updater.exe        | 10032 | 2025-05-07 07:13:56.000000 |

#### Linking Processes

```c
explorer.exe (PID: 1500)
└── cmd.exe (PID: 2200)                  ← Triggered by malicious LNK file
    └── powershell.exe (PID: 2210)       ← Downloads and executes the payload
        └── svchost.exe (PID: 2220)      ← Masquerades as a system process
            └── asyncrat.exe (PID: 2230) ← Remote Access Trojan (C2 beaconing)
```

* Show Process Tree

```bash
vol3 -f THM-WIN-001_071528_07052025.mem windows.pstree > processtree.txt
```

* Parse Process Tree

```bash
cut -d$'\t' -f1,2,3 processtree.txt
```

#### **Observations**

```bash
PID         PPID    IMAGENAME
5252        5672    WINWORD.EXE
└── 3392       5252    pdfupdater.exe    
    ├── 2576      3392    conhost.exe    
    └── 10084     3392    windows-update        
        └── 10032    10084   updater.exe            
            └── 432     10032   cmd.exe                
                ├── 4592   432     conhost.exe                
                └── 6984   432     powershell.exe
```

#### Analysis Notes and Next Steps

| WINWORD.EXE        | 5252  | 5672  | 2025-05-07 07:13:04.000000 |
| ------------------ | ----- | ----- | -------------------------- |
| pdfupdater.exe     | 3392  | 5252  | 2025-05-07 07:13:05.000000 |
| conhost.exe        | 2576  | 3392  | 2025-05-07 07:13:05.000000 |
| windows-update.exe | 10084 | 3392  | 2025-05-07 07:13:05.000000 |
| updater.exe        | 10032 | 10084 | 2025-05-07 07:13:56.000000 |
| cmd.exe            | 432   | 10032 | 2025-05-07 07:14:36.000000 |
| conhost.exe        | 4592  | 432   | 2025-05-07 07:14:36.000000 |
| powershell.exe     | 6984  | 432   | 2025-05-07 07:14:39.000000 |

***

## Digging Deeper

#### PSSCAN

```bash
vol3 -f THM-WIN-001_071528_07052025.mem windows.psscan > psscan.txt
```

* You can compare the results of `psscan` with the results of `pslist` to uncover hidden processes

```bash
           
awk '{print $1,$3}' pslist.txt | sort > pslist_processed.txt
awk '{print $1,$3}' psscan.txt | sort > psscan_processed.txt
comm -23 psscan_processed.txt pslist_processed.txt
```

#### PSXVIEW

```bash
vol3 -f THM-WIN-001_071528_07052025.mem windows.psxview > psxview.txt
```

* Filter Results

```bash
awk 'NR==3 || $4 == "False"' psxview.txt
```

***

## Dumping the Process Memory

### Finding the Path

* windows.dlllist

```bash
vol3 -f THM-WIN-001_071528_07052025.mem windows.dlllist --pid 5252 > 5252_dlllist.txt
```

```bash
cat 5252_dlllist.txt | awk '{print $1,$2,$6}'
```

#### Dump Process Memory

* windows.dumpfiles

```bash
      
mkdir PID_5252
cd PID_5252
vol3 -f ../THM-WIN-001_071528_07052025.mem windows.dumpfiles --pid 5255
```

The dumped files all follow the same naming convention: `file.StartAddress.EndAddress.ImageSectionObject.filename.img` or `file.StartAddress.EndAddress.DataSectionObject.filename.dat`&#x20;

* Find Executables

```bash
ls  | grep -E ".exe|.dat" -i
```

* Find Macro Files

```bash
ls | grep -E ".docm|.dotm" -i
```

***

### Analysis Dump files

1. **strings** → extract human-readable text (look for IPs, domains, commands, registry keys).

```
strings file.*.img | less
```

**grep** + regex → filter the output of `strings` for keywords like `.exe`, `.dll`, `http`, `cmd`&#x20;

2. file to know the file type (EXE, DLL, Data)

```
file file.*.img
```

3. **hashing (md5/sha256)** → calculate file hashes and check against VirusTotal or other threat intel sources.

```
sha256sum file.*.img
```
