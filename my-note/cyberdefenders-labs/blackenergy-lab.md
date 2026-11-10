---
description: Volatility
---

# BlackEnergy Lab

<details>

<summary><strong>Scenario</strong></summary>

A multinational corporation has suffered a cyber attack, resulting in the theft of sensitive data. The attack employed a previously unseen variant of the BlackEnergy v2 malware. The company's security team has obtained a memory dump from the infected machine and is seeking your expertise as a SOC analyst to analyze the dump in order to understand the scope and impact of the attack.

</details>

<details>

<summary><strong>Questions</strong></summary>

### Q1: Which volatility profile would be best for this machine?

<figure><img src="../../.gitbook/assets/01 (7).png" alt=""><figcaption></figcaption></figure>

```
Ans: WinXPSP2x86
```

### Q2:  How many processes were running when the image was acquired?

<figure><img src="../../.gitbook/assets/02 (9).png" alt=""><figcaption></figcaption></figure>

total process 25 with 6 terminated

```
Ans: 19
```

Q3: What is the process ID of `cmd.exe`?

<figure><img src="../../.gitbook/assets/03 (8).png" alt=""><figcaption></figcaption></figure>

```
Ans: 1960
```

### Q4: What is the name of the most suspicious process?

<figure><img src="../../.gitbook/assets/04 (4).png" alt=""><figcaption></figcaption></figure>

```
Ans: rootkit.exe
```

### Q5: Which process shows the highest likelihood of code injection?

<figure><img src="../../.gitbook/assets/06 (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/05 (5).png" alt=""><figcaption></figcaption></figure>

```
Ans: svchost.exe
```

### Q6: There is an odd file referenced in the recent process. Provide the full path of that file.

<figure><img src="../../.gitbook/assets/07 (2).png" alt=""><figcaption></figcaption></figure>

```
Ans: C:\WINDOWS\system32\drivers\str.sys
```

### Q7: What is the name of the injected DLL file loaded from the recent process?

<figure><img src="../../.gitbook/assets/08 (2).png" alt=""><figcaption></figcaption></figure>

```
Ans: msxml3r.dll
```

Q8: What is the base address of the injected DLL?

<figure><img src="../../.gitbook/assets/09 (2).png" alt=""><figcaption></figcaption></figure>

```
Ans: 0x980000
```





</details>
