---
title: "Legacy"
date: 2023-06-26T00:50:22+02:00
tags: ["smb","windows"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Legacy.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A windows room that has vulnerable samba service running."
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

- Room name: Legacy
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Legacy

![Untitled](/HTB/legacy-icon.png)

## Tools Used

- Nmap
- Msfvenom

## Port Scanning

First we need to do a port scan to know what are we trying to exploit.

![Untitled](/HTB/nmap-legacy.png)

Look like we have two smb ports the 139,445 so we are going to focus on them ...

## Enumeration

With `sudo nmap 10.10.10.14 -p 139,445 --script=vuln` I am going to see if nmap can see some vulnerabilities.

![Untitled](/HTB/smb-vuln-legacy.png)

I found two possible vulnerabilities the MS17-010 (CVE-2017-0143) and MS08-067 (CVE-2008-4250), so first I tried to pwn teh first one, but I don't know why,
it throw me a lot of errors and then I tried with the last one ...

## Exploiting

The CVE-2017-0143 or better called Eternal Blue is a vulnerability that allows us to exploit the Microsoft’s implementation of the Server Message Block (SMB) protocol with a crafted package to have RCE in the victim machine. 

![Untitled](/HTB/crackmapexec-legacy.png)

But in this case I focused on the last one called ms08-067 with metasploit and was so easy. I could do it by other programs more OSCP
firendly but I think that in the OSCP they let you do one with metasploit.

![Untitled](/HTB/ms08-legacy.png)

And we are now in :))

## Gaining an Initial Foothold

![Untitled](/HTB/root-legacy.png)

We have root permissions so the machine is finished...

## Lessons Learned

- What insights did you gain from the room?
    
    Now, we know how to perform a windows samba attack with the eternal blue vulnerability that allows me to RCE the victim machine.
    
- Were there any novel tools or techniques employed?
    
    I performed a CVE-2017-0143 exploit and msfvenom reverse shell.

**Thank you for reading, and happy hacking! 😄**
