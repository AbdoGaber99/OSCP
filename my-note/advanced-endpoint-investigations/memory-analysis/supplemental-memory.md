# Supplemental Memory

## Lateral Movement and Discovery

### Q1: The IR team suspects that the threat actor may have performed lateral movement to this host. Which executed process provides evidence of this activity?

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-19 at 14-38-59 TryHackMe Supplemental Memory.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
**Answer:** wmiprvse.exe
{% endhint %}

### Q2: What is the MITRE technique ID associated with the lateral movement method used by the threat actor?

{% hint style="success" %}
**Answer:** T1021.006
{% endhint %}

### Q3: Which other process was executed as part of the lateral movement activity to this host?



<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-19 at 14-40-49 TryHackMe Supplemental Memory.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
**Answer:** TeamsView.exe
{% endhint %}

### Q4:What is the Security Identifier (SID) of the user account under which the process was executed on this host?

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-19 at 15-18-26 TryHackMe Supplemental Memory.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Answer: S-1-5-21-3147497877-3647478928-1701467185-1008
{% endhint %}

### Q5: What is the name of the domain-related security group the user account was a member of?

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-19 at 15-17-58 TryHackMe Supplemental Memory.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Answer: Domain Users
{% endhint %}

### Q6: Which processes related to discovery activity were executed by the threat actor on this host? Format: In alphabetical order

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-19 at 15-15-50 TryHackMe Supplemental Memory (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Answer: ipconfig.exe, systeminfo.exe, whoami.exe
{% endhint %}

### Q7: What is the Command and Control IP address that the threat actor connected to from this host as a result of the previously executed actions? Format: IP Address:Port

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-19 at 15-29-13 TryHackMe Supplemental Memory.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Answer: 34.244.169.133:1995
{% endhint %}

***

## Privilege Escalation and Credential Dumping

### Q8: Conduct a deeper investigation and identify another suspicious process on the host. Provide a full path to the process in your answer.

{% hint style="success" %}
Answer: C:\Windows\Temp\pan.exe
{% endhint %}

### Q9: Which account was used to execute this malicious process?

{% hint style="success" %}
Answer: Local System
{% endhint %}

### Q10: What was the malicious command line executed by the process?

{% hint style="success" %}
Answer: privilege::debug sekurlsa::logonpasswords
{% endhint %}

### Q10: Given the command line from the previous question, which well-known hacker tool is most likely the malicious process?

{% hint style="success" %}
Answer: Mimikatz
{% endhint %}

### Q11: Which MITRE ATT\&CK technique ID corresponds to the method the attacker employed to evade detection, as identified in the previous steps?

{% hint style="success" %}
Answer:&#x20;
{% endhint %}
