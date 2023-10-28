---
title: BSides Jeddah Part 1
date: 2023-03-27 12:00:00 -500
categories: [writeups, CyberDefenders]
tags: [Wireshark, Network Traffic Analysis, Forensics]
---
# Solutions to BSidesJeddah-Part1 from Cyberdefenders.org
## Contents
1. [Question 1](#question-1)
2. [Question 2](#question-2)
3. [Question 3](#question-3)
4. [Question 4](#question-4)
5. [Question 5](#question-5)
6. [Question 6](#question-6)
7. [Question 7](#question-7)
8. [Question 8](#question-8)
9. [Question 9](#question-9)
10. [Question 10](#question-10)
11. [Question 11](#question-11)
12. [Question 12](#question-12)
13. [Question 13](#question-13)
14. [Question 14](#question-14)
15. [Question 15](#question-15)
16. [Question 16](#question-16)
17. [Question 17](#question-17)
18. [Question 18](#question-18)
19. [Question 19](#question-19)
20. [Question 20](#question-20)
21. [Question 21](#question-21)
22. [Question 22](#question-22)
23. [Question 23](#question-23)
24. [Question 24](#question-24)
25. [Question 25](#question-25)
26. [Question 26](#question-26)


# <b> 1. What is the victim's MAC address? </b> <a name = "question-1"> </a>
For this question, I opened the challenge pcap in wireshark. I began by viewing the protocol hierarchy to determine what sorts of traffic we're looking at. I noticed that there was some TLS traffic, and given that this was a phishing attack, per the challenge description, I filtered for TLS. Testing the MAC addresses of some of the hosts in the TLS stream, I found that the victim's MAC address was:
<details>
    <summary> <b>Solution </b> </summary>
    00:0c:29:b7:ca:91
</details>

# <b> 2. What is the address of the company associated with the victim's machine MAC address? </b> <a name = "question-2"> </a>

For this question, I just googled the MAC address found in the previous question. The company associated with it is VMWare. So the answer is:
<details>
    <summary> <b>Solution </b> </summary>
    3401 Hillview Avenue Palo Alto CA 94304 US
</details>

# <b> 3. What is the attacker's IP address? </b> <a name = "question-3"> </a>
After identifying the victim's MAC address earlier, I also identified the victim's IP address as 192.168.112.128, I filtered for all traffic to that IP address, and TLS traffic. I then looked at the IP addresses of the hosts in the TLS stream, and found that the attacker's IP address was:
<details>
    <summary> <b>Solution </b> </summary>
    192.168.112.128
</details>

# <b> 4. What is the IPv4 address of the DNS server used by the victim machine? </b> <a name = "question-4"> </a>
Filtering again for the victim's IP address, except this time filtering for DNS traffic as well, I found that the DNS server was:
<details>
    <summary> <b>Solution </b> </summary>
    192.168.112.2
</details>

# <b> 5. What domain is the victim looking up in packet 5648? </b> <a name = "question-5"> </a>
I simply scrolled down to packet 5648, and found that the victim was looking up the domain:
<details>
    <summary> <b>Solution </b> </summary>
    omextemplates.content.office.net
</details>

# <b> 6. What is the server certificate public key that was used in TLS session: 731300002437c17bdfa2593dd0e0b28d391e680f764b5db3c4059f7abadbb28e </b> <a name = "question-6"> </a>
Filtering the packets using 
    
    tls.handshake.session_id
yielded 2 packets. The first packet had a session ID that matched the one provided by the question. There was a field called "EC Diffie-Hellman Server Params" Within this field was the value "Pubkey" which was the public key of the server certificate.
<details>
    <summary> <b>Solution </b> </summary>
    64089e29f386356f1ffbd64d7056ca0f1d489a09cd7ebda630f2b7394e319406
</details>

# <b> 7. What domain is the victim connected to in packet 4085 </b> <a name = "question-7"> </a>
Scrolling down to pakcet 4085, I found that the protocol was TCP. I followed the TCP stream, and within it I found that the victim was connecting to the domain:
<details>
    <summary> <b>Solution </b> </summary>
    v10.vortex-win.data.microsoft.com
</details>

# <b> 8. The attacker conducted a port scan on the victim machine. How many open ports did the attacker find? </b> <a name = "question-8"> </a>

I struggled with this one. I found a [guide](https://weberblog.net/nmap-packet-capture/) that helped me to understand how to filter the TCP traffic for only those ports that were open. I then filtered for the open ports using

    (tcp.flags.ack == 1) && (tcp.flags.syn == 1)
Then just count how many unique ports there are in the resulting packets.
<details>
    <summary> <b>Solution </b> </summary>
    7
</details>


