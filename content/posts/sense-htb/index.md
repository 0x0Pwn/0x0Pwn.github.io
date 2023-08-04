---
title: "Sense"
date: 2023-06-24T23:52:47+02:00
tags: ["web","linux","pfSense"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Sense.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: ""
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

- Room name: Sense
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Sense

![Untitled](/HTB/Sense.png)

## Tools Used

- nmap
- wfuzz
- whatweb
- exploit.py


## Port Scanning

![Untitled](/HTB/nmap-sense.png)

In the port scan we can see that there are only two open ports, 443(https) and 80(http)...

## Enumeration

As always first thing i'm going to do is:

```bash
whatweb 10.10.10.60
```
But this time it throws me nothing relevant.

Then I did a dns and url scan with **wfuzz**. On first time it throws me nothing relevant and I went 
to take a look and try combinations on the website (also I tried with **hydra**), which by the way was a page of a login.

![Untitled](/HTB/login-sense.png)

After a long time trying combinations with hydra and seeing different exploits and vulnerabilities with searchsploit and
others, I began to think that I missed something and decided to do a web analysis again to discover directories in .txt .php .sh etc. 
Since in the first scan I did I got a .txt file that contained some information about vulnerabilities ...

And let's go we have user and passwd :)

![/Untitled](/HTB/credentials-sense.png)

So I logged and in the menu page said's that the version of pfSense was 2.1.3, so I search it on searchsploit and there was only a exploit,
I tried them but no works, so I searched the CVE on google and I found one that works :)

![/Untitled](/HTB/root-sense.png)

the exploit code:

![/Untitled](/HTB/exploit-sense.png)


## Escalation of privileges

There's no escalation on this machine due to I'm root at the first time. 

![Untitled](/HTB/flags-sniper.png)


## Leason Learned

- What insights did you gain from the room?
    
    I acquired knowledge on the research of sploits.
    
- Were there any novel tools or techniques employed?
    
    No.

**Thank you for reading, and happy hacking! 😄**
