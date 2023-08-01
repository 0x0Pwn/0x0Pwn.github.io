---
title: "Nineveh"
date: 2023-06-26T12:51:31+02:00
tags: ["linux","lfi"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Nineveh.png
description: "This write-up will be about the HackTheBox machine called Nineveh. "
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A linux room that " 
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/pwndside/pwndside.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Room Information

- Room name: Nineveh
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Nineveh

![Untitled](/HTB/Nineveh.png)

## Tools Used

- Nmap
- Searchsploit
- 

## Port Scanning

Other room, other port scan.

```bash
nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn 10.10.10.43 -oN escaneo 
```
![Untitled](/HTB/nmap-nineveh.png)

I found only 1 open port. Let’s focus on it.

## Enumeration

First I saw is that there are two url's one with https and one other with http. I found two diferent login page
one on each url.

![Untitled](/HTB/gobuster-nineveh.png)


## Exploiting

After trying some basic SQL injections and some basic names I decided to brute force with Hydra. With the following commands:
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form /db/index.php:"password=^PASS^&proc_login=^USER^:Incorrect password."
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form /department/login.php:"password=^PASS^&proc_login=^USER^:Invalid password."
```

![Untitled](/HTB/db_nineveh.png)

![Untitled](/HTB/hydra_nineveh.png)

When I logged into both I got these interfaces:

![Untitled](/HTB/login-nineveh.png)

![Untitled](/HTB/dbcreate-nineveh.png)

In the page of department, I saw that in **notes** menu the url changes to something so strange. So as always I tried
to find LFI and surprise there is jeje.

![Untitled](/HTB/lfi-nineveh.png)

Once I discovered this I will go to make an oo on the other website. The name of phpLiteAdimn was strange to me since it 
also had a version, so I looked it up in searchsploit and found that there were several vulnerabilities, one of them that 
matched my version said like this: if I manage to create a database and save it in .php, it should be stored with the rest 
in /var/temp/... and I create a table with a single box and enter a code in PHP (reverse shell), then I can access from 
the other page because of the LFI vuln and execute the php command.

![Untitled](/HTB/searchsploit-nineveh.png)

![Untitled](/HTB/dbcreate-nineveh.png)

And suprise we can controle terminal with that so let's do a reverse shell and ...

![Untitled](/HTB/bash-nineveh.png)

![Untitled](/HTB/pwnd-nineveh.png)


## Gaining an Initial Foothold



## Lessons Learned

- What insights did you gain from the room?
    
    It’s pretty similar to the legacy room, so I tried to replicate the methodology but with metasploit this time.
    
- Were there any novel tools or techniques employed?
    
    The use of metasploit and meterpreter helps me a lot to exploit easily this vulnerability.
