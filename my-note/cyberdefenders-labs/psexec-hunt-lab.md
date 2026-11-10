# PsExec Hunt Lab

## PsExec Hunt Lab

Analyze SMB traffic in a PCAP file using Wireshark to identify PsExec lateral movement, compromised systems, user credentials, and administrative shares.

## **Scenario**

An alert from the Intrusion Detection System (IDS) flagged suspicious lateral movement activity involving PsExec. This indicates potential unauthorized access and movement across the network. As a SOC Analyst, your task is to investigate the provided PCAP file to trace the attacker’s activities. Identify their entry point, the machines targeted, the extent of the breach, and any critical indicators that reveal their tactics and objectives within the compromised environment.

## **Questions**

**Q1:** To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?

{% hint style="info" %}
look for at statistical and conversation
{% endhint %}

<figure><img src="../../.gitbook/assets/01 (2).png" alt=""><figcaption></figcaption></figure>

```
10.0.0.130
```

***

Q2:To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?

{% hint style="info" %}
look for  at NTLM challenge authentication

`ip.addr == 10.0.0.133 && ntlmssp.challenge.target_name`
{% endhint %}

<figure><img src="../../.gitbook/assets/02 (2).png" alt=""><figcaption></figcaption></figure>

```
SALES-PC
```

***

Q3:Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?

{% hint style="info" %}
ip.dst\_host == 10.0.0.130
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-10 205243.png" alt=""><figcaption></figcaption></figure>

```
sslaes 
```

***

Q4:After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?

{% hint style="info" %}
Go to `File > Export Objects > SMB` to open a window displaying all files transferred over SMB in the capture
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-10 210300.png" alt=""><figcaption></figcaption></figure>

```
PSEXESVC.EXE
```

***

Q5: We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?

{% hint style="info" %}
Hidden share in windows

* `C$`&#x20;
* `ADMIN$`  >> C:\Windows
* `IPC$`
{% endhint %}

<figure><img src="../../.gitbook/assets/1704299580207.gif" alt="" width="563"><figcaption></figcaption></figure>

```
ADMIN$
```

***

Q6:We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?

{% hint style="info" %}
**IPC$ = Inter-Process Communication Share**
{% endhint %}

```
IPC$
```

***

Q7: Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?

{% hint style="info" %}
the connection of 10.0.0.130 ---> 10.0.0.133

&#x20;                                               \---> 10.0.0.131 (second infected machine)
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-10 213240.png" alt=""><figcaption></figcaption></figure>

```
MARKETING-PC
```
