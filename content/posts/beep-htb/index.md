---
title: "Beep"
date: 2023-06-26T12:51:31+02:00
tags: ["lfi","linux"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Beep.png
description: "This write-up will be about the HackTheBox machine called Beep. "
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A linux room that is vulnerable to a lot of things, but in this case throw LFI."
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

- Room name: Beep
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Beep

![Untitled](/HTB/Beep.png)

## Tools Used

- Nmap
- Metasploit
- Meterpreter

## Port Scanning

Other room, other port scan.

```py
 nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn 10.10.10.43 -oN escaneo
```

![Untitled](/HTB/nmap-beep.png)

I found a lot of open ports. But like always I'm focus on http and https ports ...

## Enumeration

The web page redirects to https://10.10.10.7 an throws a login panel of `elastix`

![Untitled](/HTB/elastix-beep.png) 

As always I ran a wfuzz scan to find new archives or directories ...

![Untitled](/HTB/wfuzz1-beep.png)

![Untitled](/HTB/wfuzz2-beep.png)

Nothing relevant, only that the web `https://10.10.10.7/` contains a `/cgi-bin/` folder, shellshock :)?

On the other hand, I do a searchsploit of elastix.

![Untitled](/HTB/searchsploit-beep.png)

And I found one on perl that says something of LFI, that I back comprobed that it's true.

![Untitled](/HTB/exploit-beep.png)

![Untitled](/HTB/users-beep.png)

![Untitled](/HTB/lfi-beep.png)

## Exploiting

Now we have a lot of users and passwords so let's try by SSH ...

![Untitled](/HTB/pawnd-beep.png)


## Privilege Scalation

There's no privilege escalation because I entered as root :))

## Lessons Learned

- What insights did you gain from the room?
    
    I learned about elastix and how to pawn it.
    
- Were there any novel tools or techniques employed?
    
    No
