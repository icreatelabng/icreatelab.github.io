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
