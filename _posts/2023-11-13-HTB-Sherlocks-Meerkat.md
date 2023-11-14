---
title: Sherlocks - Meerkat
date: 2023-11-13 12:00:00 -500
categories: [writeups, HackTheBox]
tags: [Sherlocks, Forensics, Network Traffic Analysis, CVE-2022-25237]
---
Woohoo!! Finally some blue-team things from HackTheBox. We love to see it. Here is a writeup for the very first one, called Meerkat. 


We're prompted with a file download that contains some alerts in json format as well as a pcap file containing all type of interesting traffic. 

#### task 1
 > We believe our Business Management Platform server has been compromised. Please can you confirm the name of the application running?

Just a quick glance at the packet details and requests, plus a quick google search will get you this one. I won't post answers here, do it yourself ;)

#### task 2
> We believe the attacker may have used a subset of the brute forcing attack category - what is the name of the attack carried out?

You can see a whole bunch of various username + password combinations being used for various users and various passwords. In this case, there is generally a single password used with each username. That corresponds to a specific type of brute force attack. 

#### task 3
> Does the vulnerability exploited have a CVE assigned - and if so, which one?

A quick google search for our Business Management Platform from task 1 plus "cve" yields us this answer. 

#### task 4
> Which string was appended to the API URL path to bypass the authorization filter by the attacker's exploit?

Digging into the CVE above, and comparing it with what we have observed in the web traffic, we can see that the PoC for the CVE involves appening a specific string to the end of a request to the API for the BMP. In this case, we can see that exact string being used in the pcap. 

#### task 5
> How many combinations of usernames and passwords were used in the credential stuffing attack?

I used a fun and exciting one liner for this one. Unfortunately there's a newline character at the end so you have to subtract one. Basically, it uses tshark to yoink all the relevant fields, grep to grab just the key/value pairs, awk to do some magic and format them as username:password, and then sort, uniq, and wc -l to count just the unique ones. 

`
└─$ tshark -r meerkat.pcap -Y 'http.request.method == "POST"' -T fields -e http.request.uri -e http.file_data | grep -E 'username=|password=' | awk -F'&' '{kv=""; for (i=1; i<=NF; i++) {split($i, arr, "="); kv = (kv == "") ? arr[2] : kv ":" arr[2]} print kv}'| sort | uniq | wc -l
`
#### task 6
> Which username and password combination was successful?

I just went with the set that appeared in the pcap last, chronologically. I figured that they stopped the attack because the credentials tested were successful. 

#### task 7
> If any, which text sharing site did the attacker utilise?

You can see multiple dns requests to this website, as well as a handful of http requests using the RCE CVE that was described above. 

#### task 8
> Please provide the file hash of the script used by the attacker to gain persistent access to our host.

If you follow one of the [redacted] links that the attacker used to access the text sharing site, you can see some of the tools used. Just wget the contents and md5sum it. 

#### task 9
> Please provide the file hash of the public key used by the attacker to gain persistence on our host.

Same as above. 

#### task 10
> Can you confirmed the file modified by the attacker to gain persistence?

Once more, this is found in the contents of the [redacted] links. 

#### task 11
Once you've identified what is going on with the persistence mechanisms grabbed from [redacted], a quick look at the MITRE ATT&CK framework, specifically the section on Persistence made this trivial. https://attack.mitre.org/tactics/TA0003/ 