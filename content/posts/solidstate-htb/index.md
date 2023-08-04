---
title: "SolidState"
date: 2023-06-26T12:51:31+02:00
tags: ["linux","James Server"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/SolidState.png
description: "This write-up will be about the HackTheBox machine called SolidState. "
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A windows room that is vulnerable to James Server."
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

- Room name: SolidState
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/SolidState

![Untitled](/HTB/SolidState.png)

## Tools Used

- Nmap
- nc
- searchsploit

## Port Scanning

Other room, other port scan.

I found 2 open ports for samba service 139 and 445 and the other ones are for msrpc. Let’s focus on samba.

## Enumeration

I am going to enumerate the possible vulnerabilities with `sudo nmap 10.10.10.14 -p 139,445 --script=vuln` 

```bash
Host script results:
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_smb-vuln-ms10-054: false
```

We have a eternal blue vulnerability MS17-010 (CVE-2017-0143) which gives us RCE in the Microsoft Samba services that are vulnerable, this time I am going to use metasploit to exploit it.

## Exploiting

![Untitled](/HTB/blue-1.png)

Upon started metasploit we got that search results, the second one fits well in this situation. I am going to set the Remote Host and the Local Host as any metasploit exploit and then running it.

## Gaining an Initial Foothold

We are in now

![Untitled](/HTB/blue-5.png)

We can’t use the dir and whoami command, but my intuition tells me that we are privileged users.

![Untitled](/HTB/blue-4.png)

![Untitled](/HTB/blue-2.png)

Let’s find ourselves the directory, If we go to the root directory C:\ we can find users directory and inside there are two interesting users the Administrator and haris. In his desktops there are the flags.

We are privilaged users, my intuition has not failed.

## Flags

**User Flag**

![Untitled](/HTB/blue-6.png)

**Root Flag**

![Untitled](/HTB/blue-7.png)

## Lessons Learned

- What insights did you gain from the room?
    
    It’s pretty similar to the legacy room, so I tried to replicate the methodology but with metasploit this time.
    
- Were there any novel tools or techniques employed?
    
    The use of metasploit and meterpreter helps me a lot to exploit easily this vulnerability.
