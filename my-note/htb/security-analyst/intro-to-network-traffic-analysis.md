# Intro to Network Traffic Analysis

### Traffic Captures with TcpDump

> <mark style="color:purple;">TCPDump will resolve IPs to hostnames by default</mark>

* **Basic Capture Options**

<table data-header-hidden><thead><tr><th width="161" align="center"></th><th></th></tr></thead><tbody><tr><td align="center"><strong>Switch Command</strong></td><td><strong>Result</strong></td></tr><tr><td align="center">D</td><td>Will display any interfaces available to capture from.</td></tr><tr><td align="center">i</td><td>Selects an interface to capture from. ex. -i eth0</td></tr><tr><td align="center">n</td><td>Do not convert addresses (i.e., host addresses, port numbers, etc.) to names.</td></tr><tr><td align="center">e</td><td>Will grab the ethernet header along with upper-layer data.</td></tr><tr><td align="center">X</td><td>Show Contents of packets in hex and ASCII.</td></tr><tr><td align="center">XX</td><td>Same as X, but will also specify ethernet headers. (like using Xe)</td></tr><tr><td align="center">v, vv, vvv</td><td>Increase the verbosity of output shown and saved.</td></tr><tr><td align="center">c</td><td>Grab a specific number of packets, then quit the program.</td></tr><tr><td align="center">s</td><td>Defines how much of a packet to grab.</td></tr><tr><td align="center">S</td><td>change relative sequence numbers in the capture display to absolute sequence numbers. (13248765839 instead of 101)</td></tr><tr><td align="center">q</td><td>Print less protocol information.</td></tr><tr><td align="center">r file.pcap</td><td>Read from a file.</td></tr><tr><td align="center">w file.pcap</td><td>Write into a file</td></tr></tbody></table>

***

### Filtering and Advanced Syntax Options

<table data-header-hidden><thead><tr><th width="149">Filter</th><th>Result</th></tr></thead><tbody><tr><td>host</td><td><code>host</code> will filter visible traffic to show anything involving the designated host. Bi-directional</td></tr><tr><td>src / dest</td><td><code>src</code> and <code>dest</code> are modifiers. We can use them to designate a source or destination host or port.</td></tr><tr><td>net</td><td><code>net</code> will show us any traffic sourcing from or destined to the network designated. It uses / notation.</td></tr><tr><td>proto</td><td>will filter for a specific protocol type. (ether, TCP, UDP, and ICMP as examples)</td></tr><tr><td>port</td><td><code>port</code> is bi-directional. It will show any traffic with the specified port as the source or destination.</td></tr><tr><td>portrange</td><td><code>portrange</code> allows us to specify a range of ports. (0-1024)</td></tr><tr><td>less / greater "&#x3C; >"</td><td><code>less</code> and <code>greater</code> can be used to look for a packet or protocol option of a specific size.</td></tr><tr><td>and / &#x26;&#x26;</td><td><code>and</code> <code>&#x26;&#x26;</code> can be used to concatenate two different filters together. for example, src host AND port.</td></tr><tr><td>or</td><td><code>or</code> allows for a match on either of two conditions. It does not have to meet both. It can be tricky.</td></tr><tr><td>not</td><td><code>not</code> is a modifier saying anything but x. For example, not UDP.</td></tr></tbody></table>

Examples

```shellscript
sudo tcpdump -i eth0 host 172.16.146.2              ##Host Filter
sudo tcpdump -i eth0 src host 172.16.146.2          #Source/Destination Filter
sudo tcpdump -i eth0 tcp src port 80                #Utilizing Source With Port as a Filter
sudo tcpdump -i eth0 dest net 172.16.146.0/24       #Using Destination in Combination with the Net Filter
sudo tcpdump -i eth0 udp                            #Protocol Filter - Common Name
sudo tcpdump -i eth0 proto 17                       #Protocol Filter - Number
sudo tcpdump -i eth0 tcp port 443                   #Port Filter
sudo tcpdump -i eth0 portrange 0-1024               #Port Range Filter
sudo tcpdump -i eth0 less 64                        #Less/Greater Filter
sudo tcpdump -i eth0 greater 500
sudo tcpdump -i eth0 host 192.168.0.1 and port 23   #AND Filter
sudo tcpdump -r sus.pcap icmp or host 172.16.146.1  #OR Filter
```
