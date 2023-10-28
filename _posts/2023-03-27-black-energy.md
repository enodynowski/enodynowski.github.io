---
title: Black Energy
date: 2023-03-27 12:00:00 -500
categories: [writeups, CyberDefenders]
tags: [volatility, memory forensics, Forensics]
---
# Solutions to BlackEnergy from Cyberdefenders.org
## Contents

## <b> 1. Which volatility profile would be best for this machine?</b> 
    vol2 imageinfo -f BlackEnergy.raw

![imageinfo](/assets/images/blackenergy/imageinfo.png){: .shadow } 
<details>
    <summary> <b>Solution </b> </summary>
        WinXPSP2x86
</details>

## <b> 2. How many processes were running when the image was acquired?</b> 
To determine the quantity of proceses that were running at the time the image was acquired, run:

    vol2 -f BlackEnergy.raw --profile=WinXPSP2x86 pslist

Then discount the processes that were exited at some point.
![pslist](/assets/images/blackenergy/pslist.png){: .shadow }

<details>
    <summary> <b>Solution </b> </summary>
        19
</details>

## <b> 3. What is the process ID of cmd.exe? </b> 
The process ID of cmd.exe is also found in the above output of the pslist plugin for volatility 2
<details> 
    <summary> <b>Solution </b> </summary>
    1960
</details>

## <b> 4. What is the name of the most suspicious process? </b> 
The most suspicious process can easily be found in the above output of the pslist plugin for volatility 2
<details>
    <summary> <b>Solution </b> </summary>
    rootkit.exe
</details>

## <b> 5. Which process shows the highest likelihood of code injection? </b> 
Continuing to use the output found above in the pslist plugin, it is clear which process is likely the subject of code injection. 

<details>
    <summary> <b>Solution </b> </summary>
        svchost.exe
</details>

## <b> 6. There is an odd file referenced in the recent process. Provide the full path of that file. </b> 
Since it is unclear which of the svchost.exe processes is the one to be investigated, I took note of each of their PID's for use in this step. Then, using the command: 

        vol2 -f BlackEnergy.raw --profile=WinXPSP2x86 -p (PID) handles -t file

I looked at all of the files that each of the processes interacts with. PID 880 looks promising. 
![filehandles](/assets/images/blackenergy//filehandles.png){: .shadow }
<details>  
    <summary> <b>Solution </b> </summary>
    C:\WINDOWS\system32\drivers\str.sys
</details>

## <b> 7. What is the name of the injected dll file loaded from the recent process? </b> 
There are a few ways to see the loaded dll's of a specific process, however not all of them prove to be fruitful here. For example, the dlllist plugin does not indicate which of these dll's are linked to the ldr modules and which are not. Additionally, the volatility 2 documentation indicates that the ldrmodules plugin is more suited for this purpose. 

Using the command:

    vol2 -f BlackEnergy.raw --profile=WinXPSP2x86 -p 880 ldrmodules

We can see all of the dll's that are associated with out malicious process. Of them, only one is not linked to the 3 ldr modules, making it the most suspicious. 
![ldrmodules](/assets/images/blackenergy//ldrmodules.png){: .shadow }
<details>
    <summary> <b>Solution </b> </summary>
        msxml3r.dll
</details>

## <b> 8. What is the base address of the injected dll? </b>
To get the base address of the injected dll, you can use the malfind plugin in volatility. This provides a bunch of useful information regarding potentially malicious or injected code in a specified process. 

    vol2 -f BlackEnergy.raw --profile=WinXPSP2x86 -p 880 malfind

![malfind](/assets/images/blackenergy//malfind.png){: .shadow }
<details>
    <summary> <b>Solution </b> </summary>
    0x980000
</details>
