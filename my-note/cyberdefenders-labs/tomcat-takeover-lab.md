# Tomcat Takeover Lab

## **Scenario**

The SOC team has identified suspicious activity on a web server within the company's intranet. To better understand the situation, they have captured network traffic for analysis. The PCAP file may contain evidence of malicious activities that led to the compromise of the Apache Tomcat web server. Your task is to analyze the PCAP file to understand the scope of the attack

## **Questions**

Q1: Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

{% hint style="info" %}
In Wireshark, go to `Statistics > Conversations` and select the TCP tab. Review the list of conversations
{% endhint %}

<figure><img src="../../.gitbook/assets/04 (2).png" alt=""><figcaption></figcaption></figure>

```
14.0.0.120
```

***

Q2: Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?

{% hint style="info" %}
[https://whois.domaintools.com/14.0.0.120](https://whois.domaintools.com/14.0.0.120)
{% endhint %}

<figure><img src="../../.gitbook/assets/05 (2).png" alt=""><figcaption></figcaption></figure>

```
china
```

***

Q3: From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?

{% hint style="info" %}
follow http stream
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-11 184616.png" alt=""><figcaption></figcaption></figure>

```
8080
```

***

Q4: Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

{% hint style="info" %}
tools

* Gobuster
* DirBuster
* ffuf
* dirsearch
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-11 185357.png" alt=""><figcaption></figcaption></figure>

```
Gobuster
```

***

Q5: After the effort to enumerate directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which specific directory related to the admin panel did the attacker uncover?

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-11 190348.png" alt=""><figcaption></figcaption></figure>

```
/manager
```

***

Q6: After accessing the admin panel, the attacker tried to brute-force the login credentials. Can you determine the correct username and password that the attacker successfully used for login?

{% hint style="info" %}
filter on `http.authbasic`
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-11 193733.png" alt=""><figcaption></figcaption></figure>

```
admin:tomcat
```

***

Q7:Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?



<figure><img src="../../.gitbook/assets/Screenshot 2026-04-11 193904.png" alt=""><figcaption></figcaption></figure>

```
JXQOZY.war
```

***

Q8: After successfully establishing a reverse shell on our server, the attacker aimed to ensure persistence on the compromised machine. From the analysis, can you determine the specific command they are scheduled to run to maintain their presence?

{% hint style="info" %}
follow http stream at upload request and follow up
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-11 194259.png" alt=""><figcaption></figcaption></figure>

> JXQOZY.war
