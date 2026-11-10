# Information Gathering

## Passive Information Gathering

### 1. Whois Enumeration

```
whois megacorpone.com
whois 38.100.193.70
```

### 2. Google Hacking

ref

* [https://dorksearch.com/](https://dorksearch.com/)
* [https://www.exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database)

```
site:megacorpone.com
site:megacorpone.com -filetype:txt
intitle:“index of” “parent directory”
```

<details>

<summary>Google Dorks</summary>

## 🕵️ Google Dorking Cheat Sheet for Recon and CTF

### 📁 Open Directories and File Listings

| Dork                                        | Description                                     |
| ------------------------------------------- | ----------------------------------------------- |
| `intitle:"index of"`                        | Lists open directories with no index protection |
| `intitle:"index of" "parent directory" mp3` | Open directories containing MP3 files           |
| `intitle:"index of" site:example.com`       | Open directories within a specific domain       |

***

### 🔐 Passwords and Sensitive Data

| Dork                           | Description                               |
| ------------------------------ | ----------------------------------------- |
| `intext:"password"`            | Pages containing the word "password"      |
| `filetype:log intext:password` | Log files that contain passwords          |
| `filetype:env DB_PASSWORD`     | `.env` config files exposing DB passwords |

***

### ⚙️ Configuration Files

| Dork                           | Description                             |
| ------------------------------ | --------------------------------------- |
| `filetype:conf apache`         | Apache configuration files              |
| `filetype:ini intext:password` | INI config files with exposed passwords |
| `filetype:xml intext:password` | XML files containing credentials        |

***

### 🔧 Admin Panels and Login Pages

| Dork                           | Description                        |
| ------------------------------ | ---------------------------------- |
| `intitle:"admin login"`        | Admin login pages                  |
| `inurl:admin`                  | URLs with the word "admin"         |
| `site:example.com inurl:admin` | Admin pages within a specific site |

***

### 📑 Documents and Leaked Data

| Dork                            | Description                             |
| ------------------------------- | --------------------------------------- |
| `filetype:pdf site:example.com` | PDF files from a target domain          |
| `filetype:xls intext:email`     | Excel sheets containing email addresses |
| `filetype:sql intext:password`  | SQL dumps with exposed passwords        |

***

### 🎯 Domain-Specific Targeting

| Dork                                   | Description                            |
| -------------------------------------- | -------------------------------------- |
| `site:example.com`                     | Limit searches to a specific domain    |
| `site:*.gov filetype:pdf confidential` | Government PDFs marked as confidential |

***

### ⚠️ Legal Notice

> Always use Google Dorks ethically and legally.\
> Use only in:
>
> * Penetration testing engagements with permission
> * Capture The Flag (CTF) competitions
> * Your own lab environments

***

✅ Tip: Combine dorks creatively to narrow your results!

</details>

### 3. Netcraft

{% embed url="https://searchdns.netcraft.com/" %}

### 4. Open-Source Code

* github.com

```
site:github.com inurl:megacorpone
user:megacorpone filename:users
```

* gitlap.com
* SourceForge

tools

{% embed url="https://github.com/michenriksen/gitrob" %}

{% embed url="https://github.com/gitleaks/gitleaks" %}

<figure><img src="../../.gitbook/assets/01 (16).png" alt=""><figcaption></figcaption></figure>

### 5. Shodan

### 6. Security Headers and SSL/TLS

{% embed url="https://securityheaders.com" %}

<figure><img src="../../.gitbook/assets/02 (18).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.ssllabs.com/ssltest/" %}

<figure><img src="../../.gitbook/assets/03 (16).png" alt=""><figcaption></figcaption></figure>

***

## Active Information Gathering

### <mark style="color:green;">DNS Enumeration</mark>

tools

* host
* dig
* dnsrecon
* nslookup
* dnsenum&#x20;

host command to find the IP address

```bash
host www.domain.com
```

MX or TXT records

```bash
host -t mx domain.com
host -t txt domain.com
```

use word list

```bash
for ip in range $(cat subdomain.txt); do host $ip.domain.com; done
```

**PTR** records for the domain

```bash
for ip in $(seq 1 254); do host x.x.x.$ip; done | grep -v "not
found"
```

DNS Zone Transfer

```bash
host -l domain.com ns1.domain.com
```

#### automate DNS enumeration

```bash
dnsrecon -d domain.com -t std
```

The `-t std` option in `dnsrecon` performs standard DNS enumeration, which includes:

* Querying common DNS records: `A`, `AAAA`, `CNAME`, `NS`, `MX`, `SOA`
* Attempting zone transfers (`AXFR`) from name servers
* Reverse DNS lookups (`PTR` records)
* (Optional) Subdomain brute-forcing if a wordlist is specified with `-D`

> If zone transfer is successful, it can reveal the full DNS zone data.

```bash
dnsrecon -d domain.com -D ~/list.txt -t brt
```

* `-t brt`: Brute-force subdomain discovery mode

```bash
nslookup mail.domain.com
```

```bash
dnsenum domain.com
```

***

### <mark style="color:green;">TCP/UDP Port Scanning</mark>

#### TCP Scan

```bash
sudo nmap -sT <IP> -p-
```

* -p- (all port)

#### UDP Scan&#x20;

```bash
sudo nmap -sU <IP> --top-ports=20
```

#### SYN Scan

```bash
sudo nmap -sS <IP> -p80
```

* -p80 (PORT 80)

#### network sweeping

```bash
sudo nmap -sn <IP> -oG ping-sweeping.txt
```

* -oG file.txt (output)

#### OS fingerprint

```bash
sudo nmap -O <IP> --osscan-guess
```

#### Banner Grabbing/Service Enumeration

```bash
sudo nmap -sV -A <IP>
```

* -A run some script

#### Nmap Scripting Engine (NSE)

* /usr/share/nmap/scripts

```bash
sudo nmap <IP> --script=smb-os-discovery
```

***

### <mark style="color:green;">SMB Enumeration</mark>

SMB work at port 445 and 139 over NetBios session service

**SMB 1.0 --> 139**

* Null Sessions (anonymous access)
* IPC$ Access&#x20;
* SMB Relay

**SMB v2 v3 ---> 445**

```bash
nmap -v -p 139,445 <IP> -oG smb.txt
nmap -v -p 139, 445 --script=smb-os-discovery <IP>
nmap -v -p 139,445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 <IP>
```

```bash
sudo nbtscan -r <IP>
```

***

### <mark style="color:green;">SMTP Enumeration</mark>

1. **User Enumeration by using VRFY (**&#x62A;قدر تستخدمهم بعد كده ف&#x64A;**)**
   * Brute-force
   * Password spraying
   * Spear phishing
   * Lateral movement

```bash
nc <IP> 25
telnet <IP> 25

VRFY root
252 2.0.0 root → يعني اليوزر موجود ✅
550 5.1.1 → not exist
```

***

### <mark style="color:green;">SNMP Enumeration</mark>

1. work at UDP port 161

#### community strings

* `public`  Read-Only
* `private`  Read/Write

#### MIB (Management Information Base)

* System Processes (1.3.6.1.2.1.25.1.6.0)
* Running Programs (1.3.6.1.2.1.25.4.2.1.2)
* Processes Path (1.3.6.1.2.1.25.4.2.1.4)
* Storage Units (1.3.6.1.2.1.25.2.3.1.4)
* Software Name (1.3.6.1.2.1.25.6.3.1.2)
* User Accounts (1.3.6.1.4.1.77.1.2.25)
* TCP Local Ports (1.3.6.1.2.1.6.13.1.3)

#### Scanning for SNMP (nmap)

```bash
sudo nmap -sU --open -p 161 <IP>
```

#### brute force attack (onesixtyone )

```bash
echo public > community
echo private >> community
echo manager >> community
for ip in $(seq 1 254); do echo 192.168.50.$ip; done > ips
onesixtyone -c community -i ips
```

#### Enumerating the Entire MIB Tree

* all MIB

```bash
snmpwalk -c public -v1 -t 10 10.11.1.14
```

* Windows Users

```bash
snmpwalk -c public -v1 <IP> 1.3.6.1.4.1.77.1.2.25
```

* Running Windows Processes

```bash
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.4.2.1.2
```



