# Investigating Network Scans

### Task 1: Examine the First Captured File Named 1.pcapng

Let's load the first provided PCAP file \[located at **Desktop/Module7/Lab21/1.pcapng**] in Wireshark.

We know that most types of TCP scans start with a SYN message.

An attacker conducting Port scanning will likely be sending a large number of SYN segments. We can filter for TCP messages with SYN Flags in Wireshark in order to identify the IP address of the attacker.

<figure><img src="https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/1.png" alt=""><figcaption></figcaption></figure>

After using the proper filter, we can see that most of the SYN segments are coming from the IP: **192.168.153.133**

In order to determine the type of scan, we need to examine at least one port being scanned. We can filter the traffic looking for segments going or coming from a certain port and examine how this port was scanned.

For this example will examine how Port 80 was scanned. Using filter `tcp.port==80`

<figure><img src="https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/2.png" alt=""><figcaption></figcaption></figure>

We can see that the attacker has completed the three-way handshake before sending the RST segment to end the connection.

This suggests that this is a full TCP scan.

We can also verify that Port 80 was opened on the machine that was being scanned because the victim machine replied with SYN ACK message.

To examine more open ports we need to look for SYN-ACK messages which originated from the victim machine.

Using the proper Wireshark filter we can get all the messages which originated from open ports

```
(tcp.ack==1 and tcp.flags.syn==1) and ip.src==192.168.153.130
```

<figure><img src="https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/3.png" alt=""><figcaption></figcaption></figure>

### Task 2: Examine the Second Captured File Named 2.pcapng

Let's now load the second provided PCAP file \[located at **Desktop/Module7/Lab21/2.pcapng**] in Wireshark.

To examine the differences between this type of scan and the previous one, we need to examine how at least one port was scanned.

Let's examine port 445 which is usually opened by default on most windows systems. Using filter `tcp.port==445`

<figure><img src="https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/4.png" alt=""><figcaption></figcaption></figure>

We can see that the attacker terminated the connection immediately after receiving the SYN, ACK message from the client.

This suggests that this is a half-open scan where the attacker 192.168.153.133 is scanning 192.168.153.130.

Same as before, we can examine the open ports by filtering the messages which originated from the server and have both the SYN and ACK flags set.

<figure><img src="https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/5.png" alt=""><figcaption></figcaption></figure>

### Task 3: Examine the Third Captured File Named Idle.pcapng

Let's load the third provided PCAP file \[located at **Desktop/Module7/Lab21/idle.pcapng**] in Wireshark.

From the first look at it, this seems like the other typical scans we have examined earlier. We can use Wireshark filters to review the syn segments being sent

```
tcp.flags.syn==1 and tcp.flags.ack==0
```

![Content Image](https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/6.png)

We can see that the machine with IP 192.168.153.140 seems to be scanning the 192.168.153.130 machine.

From the first look at the transport layer headers there doesn't seem to be anything wrong just a typical half-open scan being conducted and the opened ports are replying with SYN, ACK messages.

```
tcp.flags.syn==1 and tcp.flags.ack==1 and ip.src==192.168.153.130
```

<figure><img src="https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/7.png" alt=""><figcaption></figcaption></figure>

The only noticeable difference is the number of repeated segments being sent which we haven't seen when we examined the half-open scan in the previous task.

### Task 4: Find a Way to Make Sure that the Identified IP is Actually the One of the Attacker

If what the owner of the 192.168.153.140 machine is saying is true, then there is someone forging packets, putting his address on it, and sending it on its behalf.

One way to prove that an impersonator exists in the network is to find two packets with the same Source IP address on them (zombie's IP) but with different machine-related headers such as the TTL and the MAC address.

Basically, if we find two packets with the same Source IP address and different Mac addresses, this means that there are two different sources for those packets.

Let's first filter and examine the SYN segments which have the zombie's IP address on it.

```
ip.src==192.168.153.140
```

![Content Image](https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/8.png)

We'll examine two different segments (a SYN and a RST), frames 56 and 57 to verify whether they have been sent from the same machine or not.

![Content Image](https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/9.png)

By listing the headers of the two messages, we can clearly see that there are many differences in machine-related fields between the two packets.

First, even though the two packets have the same Source IP, they have a different source MAC address and different TTL, which is more than enough to suggest that the two packets originated from different network interface cards.

In order to find the true attacker behind this, we can use the MAC addresses we got to search the packet file for packets that have the same MAC but different IP addresses. Those packets should belong to the true attacker's machine.

Since we know by now that this is a "zombie scan" attempt, there is no need to try and lookup both Mac addresses.

We know that the first step in a "zombie scan," is when the attacker forges a SYN message with the zombies Source IP address on it.

So the MAC address on the forged SYN message the victim has received is the attacker's true Mac address.

To easily find the packet we're looking for, we can filter and look for packets which have the source Mac address we're looking for and doesn't have the zombie's IP address.

```
eth.src==00:0c:29:20:bc:14 and !(ip.addr==192.168.153.140)
```

![Content Image](https://assets.ine.com/content/ptp/lab_21_investigating_network_scans/10.png)

It seems that the true attackers IP address is 192.168.153.133.
