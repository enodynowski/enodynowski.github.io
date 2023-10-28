---
title: BSides Jeddah Part 1
date: 2023-03-27 12:00:00 -500
categories: [writeups, CyberDefenders]
tags: [Wireshark, Network Traffic Analysis, Forensics]
---
# Solutions to BSidesJeddah-Part1 from Cyberdefenders.org

# <b> 1. What is the victim's MAC address? </b> 
For this question, I opened the challenge pcap in wireshark. I began by viewing the protocol hierarchy to determine what sorts of traffic we're looking at. I noticed that there was some TLS traffic, and given that this was a phishing attack, per the challenge description, I filtered for TLS. Testing the MAC addresses of some of the hosts in the TLS stream, I found that the victim's MAC address was:
<details>
    <summary> <b>Solution </b> </summary>
    00:0c:29:b7:ca:91
</details>

# <b> 2. What is the address of the company associated with the victim's machine MAC address? </b> 

For this question, I just googled the MAC address found in the previous question. The company associated with it is VMWare. So the answer is:
<details>
    <summary> <b>Solution </b> </summary>
    3401 Hillview Avenue Palo Alto CA 94304 US
</details>

# <b> 3. What is the attacker's IP address? </b> 
After identifying the victim's MAC address earlier, I also identified the victim's IP address as 192.168.112.128, I filtered for all traffic to that IP address, and TLS traffic. I then looked at the IP addresses of the hosts in the TLS stream, and found that the attacker's IP address was:
<details>
    <summary> <b>Solution </b> </summary>
    192.168.112.128
</details>

# <b> 4. What is the IPv4 address of the DNS server used by the victim machine? </b> 
Filtering again for the victim's IP address, except this time filtering for DNS traffic as well, I found that the DNS server was:
<details>
    <summary> <b>Solution </b> </summary>
    192.168.112.2
</details>

# <b> 5. What domain is the victim looking up in packet 5648? </b> 
I simply scrolled down to packet 5648, and found that the victim was looking up the domain:
<details>
    <summary> <b>Solution </b> </summary>
    omextemplates.content.office.net
</details>

# <b> 6. What is the server certificate public key that was used in TLS session: 731300002437c17bdfa2593dd0e0b28d391e680f764b5db3c4059f7abadbb28e </b> 
Filtering the packets using 
    
    tls.handshake.session_id
yielded 2 packets. The first packet had a session ID that matched the one provided by the question. There was a field called "EC Diffie-Hellman Server Params" Within this field was the value "Pubkey" which was the public key of the server certificate.
<details>
    <summary> <b>Solution </b> </summary>
    64089e29f386356f1ffbd64d7056ca0f1d489a09cd7ebda630f2b7394e319406
</details>

# <b> 7. What domain is the victim connected to in packet 4085 </b> 
Scrolling down to pakcet 4085, I found that the protocol was TCP. I followed the TCP stream, and within it I found that the victim was connecting to the domain:
<details>
    <summary> <b>Solution </b> </summary>
    v10.vortex-win.data.microsoft.com
</details>

# <b> 8. The attacker conducted a port scan on the victim machine. How many open ports did the attacker find? </b> 

I struggled with this one. I found a [guide](https://weberblog.net/nmap-packet-capture/) that helped me to understand how to filter the TCP traffic for only those ports that were open. I then filtered for the open ports using

    (tcp.flags.ack == 1) && (tcp.flags.syn == 1)
Then just count how many unique ports there are in the resulting packets.
<details>
    <summary> <b>Solution </b> </summary>
    7
</details>


