# Volatility Essentials

### Architectural Overview

* **Memory layers**: These layers represent the hierarchy of address spaces, from raw memory to virtual address translations.
* **Symbol tables**: The tables enable kernel and process structures to be interpreted through OS-specific debugging symbols.
* **Plugins**: These are modular routines that leverage the underlying memory layers and symbol tables to extract artefacts of forensic interest. Later in the room, we shall look at some key plugins used.

### System Requirements and Installation

```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
python3 vol.py -h
```

### Memory Acquisition Tools

#### For Windows systems

* **DumpIt:** captures a full physical memory image on 32/64‑bit Windows and automatically hashes the output.
* **WinPmem:** Open‑source driver‑based tool that acquires RAM in RAW/ELF formats and embeds acquisition metadata for chain‑of‑custody.
* **Magnet RAM Capture:** GUI‑driven collector that snapshots volatile memory on live Windows hosts while minimising footprint.
* **FTK Imager:** The most common commercial tool that acquires memory and selected logical artefacts alongside disk imaging functions.&#x20;

#### For Linux and macOS systems

* **AVML:** Lightweight Microsoft CLI utility that dumps Linux memory to a compressed ELF file without requiring a kernel module.
* **LiME:** Loadable Kernel Module for Linux that captures full volatile memory over disk or network and supports ARM/x86 architectures.
* **OSXPmem:** macOS‑specific fork of Pmem that creates raw memory images on Intel‑based Macs for subsequent Volatility analysis

#### Extracting memory from virtual environments

* VMware - `.vmem`
* Hyper-V - `.bin`
* Parallels - `.mem`
* VirtualBox - `.sav` It is worth noting that this is a partial memory file.

### Memory Analysis

```bash
python3 vol.py -f ~/Investigation-1.vmem <Plugins>
```

| Plugins            | description                            |
| ------------------ | -------------------------------------- |
| windows.info       | Windows Information                    |
| linux.info         | Linux Information                      |
| windows.pslist     | Active Process Enumeration             |
| windows.psscan     | Hidden Process Enumeration             |
| windows.pstree     | Process Hierarchy Enumeration          |
| windows.handles    | File, Registry, and Thread Enumeration |
| windows.netstat    | Network Connection Enumeration         |
| windows.netscan    | TCP/UDP Socket Enumeration             |
| windows.dlllist    | DLL Enumeration                        |
| windows.malfind    | Malware Analysis                       |
| windows.vadinfo    | Malware Analysis                       |
| windows.ssdt       | SSDT Hook Detection                    |
| windows.modules    | Kernel Module Enumeration              |
| windows.driverscan | Driver Scanning                        |





```
python3 vol.py -f Investigation-1.vmem windows.dlllist --pid <PID>
```



```
python3 vol.py -f Investigation-1.vmem windows.handles --pid <PID>
```



```
python3 vol.py -f Investigation-1.vmem -o dumps/ windows.dumpfiles --pid <PID>
```

