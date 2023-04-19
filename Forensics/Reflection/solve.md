# Hackthebox - Forensics - Reflection
### You and Miyuki have succeeded in dis-empowering Draeger's army in every possible way. Stopped their fuel-supply plan, arrested their ransomware gang, prevented massive phishing campaigns and understood their tactics and techniques in depth. Now it is the time for the final blow. The final preparations are completed. Everyone is in their stations waiting for the signal. This mission can only be successful if you use the element of surprise. Thus, the signal must remain a secret until the end of the operation. During some last-minute checks you notice some weird behaviour in Miyuki's PC. You must find out if someone managed to gain access to her PC before it's too late. If so, the signal must change. Time is limited and there is no room for errors.
#### We're given a memory dump to analyze with volatility.
#### Let's grab the windows profile we're going to use.
<img width="877" alt="1" src="https://user-images.githubusercontent.com/73375576/232719783-f7550a89-c272-48d0-908d-e5e6c96b4553.png">

#### Running pslist, pstree, psscan plugin on volatility to identify processes. We see two files that shouldn't be there on a windows preconfigured pc.
<img width="790" alt="2 1" src="https://user-images.githubusercontent.com/73375576/232719795-d8a3bbca-87d5-4674-b7b5-e67353158d44.png">
<img width="818" alt="2 2" src="https://user-images.githubusercontent.com/73375576/232719811-94a4eaea-7283-4081-9f11-9f184322792f.png">
<img width="678" alt="2 3" src="https://user-images.githubusercontent.com/73375576/232719837-8cf1e8f6-bd7c-4edc-bdf1-e2062cce419f.png">

#### I identified "powershell.exe" and "notepad.exe" as probably malicious, let's run netscan plugin to search for anomalous network activity.
<img width="1149" alt="netscan3" src="https://user-images.githubusercontent.com/73375576/232722664-7b600e42-dcb7-4c33-b830-a9059bc0f1e8.png">

#### We can use the consoles plugin to return the output of anything that was in a terminal, and what command line arguments were run. Let's do this for "powershell.exe" file.
<img width="1098" alt="4" src="https://user-images.githubusercontent.com/73375576/232725911-287a1407-ee96-4559-a794-77af309c2e58.png">

#### It appears that the user has run a powershell script called "update.ps1", let's locate it with filescan plugin.
<img width="994" alt="5" src="https://user-images.githubusercontent.com/73375576/232725923-a811fd26-b7c0-4d49-9ea6-1a01c944df72.png">

#### Let's dump the process and inspect it.
<img width="1159" alt="6" src="https://user-images.githubusercontent.com/73375576/232725932-d6d736cf-28bc-4208-a3ec-f0683e16960b.png">

#### It appears we’ve finally found the bad thing. The "windowsliveupdater.com" is not a real Microsoft domain and simply redirects us to a Rick Roll. But, here it’s been used to host a "sysdriver.ps1" file which has probably been executed in memory.
<img width="744" alt="7" src="https://user-images.githubusercontent.com/73375576/232725933-10e6b0b6-12c4-4854-96a6-58d2375b2d3d.png">

#### The cmdlet Invoke-ReflectivePEInjection has been run.

#### Some googling shows that it’s part of the PowerSploit suite of tools.
<img width="1043" alt="Screenshot 2023-04-18 at 13 32 00" src="https://user-images.githubusercontent.com/73375576/232752417-978805b4-04f3-4079-8475-4c64100a79ed.png">

#### Okay so we know what to look for, it's a reflective DLL injection.
#### • A DLL injection is a technique used for running code within the address space of another process by forcing it to load a dynamic-link library.
#### • A reflective DLL injection is a technique that allows an attacker to inject a DLL's into a victim process from memory rather than disk.

#### I used vaddump plugin to dump again the memory process of "powershell.exe". And this works more nicely because we're dumping virtual pages, using this plugin reduces a lot cleanup work.
<img width="887" alt="9" src="https://user-images.githubusercontent.com/73375576/232800832-38b1e626-5344-4e41-9e7c-b801983e8f08.png">

#### Now we're looking for the dump with the DLL inside it.
<img width="884" alt="10" src="https://user-images.githubusercontent.com/73375576/232800867-5d88d9dd-2a96-4718-9516-3c2be5bb1ddf.png">

#### Let's open that file in a hex viewer xxd and grep again the DLL.
<img width="621" alt="Screenshot 2023-04-18 at 17 47 51" src="https://user-images.githubusercontent.com/73375576/232823162-be1c99fa-8a94-40d0-b0b8-adeacfd32973.png">

#### Adding some extra lines to our grep will reveal the magic bytes on top.
<img width="524" alt="Screenshot 2023-04-18 at 18 15 55" src="https://user-images.githubusercontent.com/73375576/232824029-391bf3fa-0672-49d0-a949-c7cade226ccc.png">

#### This script will read the dump file and rebuild the DLL inside "output.dll".
<img width="556" alt="12" src="https://user-images.githubusercontent.com/73375576/232832506-eccac3dd-99f9-4e03-8023-e805c1b90fa5.png">

#### Decompiling the DLL in ghidra and looking through VoidFunc we see a lot of bytes/characters that might be concatenated and called by WinExec.
<img width="1498" alt="Screenshot 2023-04-18 at 18 56 42" src="https://user-images.githubusercontent.com/73375576/232834524-b9cd8e4e-7b83-42ef-83c3-63e15aaf2162.png">

#### I trimmed the values so we can directly copy the hex values into CyberChef.
<img width="544" alt="Screenshot 2023-04-18 at 19 11 56" src="https://user-images.githubusercontent.com/73375576/232838918-d492d583-e125-440f-8c06-d215e0d6e8d6.png">

#### Notice that 2 values are in decimal and should be replaced. The hexadecimal value of 99 is 0x63. 
<img width="679" alt="15" src="https://user-images.githubusercontent.com/73375576/232836761-318e6a01-39be-4375-9a5c-9426bafefe89.png">

#### And get a clean base64 string to decode.
<img width="693" alt="Screenshot 2023-04-18 at 19 21 54" src="https://user-images.githubusercontent.com/73375576/232841730-5cff317a-c48f-4b24-b6cb-58afd1d1a45d.png">

#### Flag here !
<img width="879" alt="16" src="https://user-images.githubusercontent.com/73375576/232841754-06a0b571-61a0-48eb-a58c-8dc943dc208e.png">

