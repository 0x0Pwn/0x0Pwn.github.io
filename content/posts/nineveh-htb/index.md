---
title: "Nineveh"
date: 2023-06-26T12:51:31+02:00
tags: ["smb",""]
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
- Metasploit
- Meterpreter

## Port Scanning

Other room, other port scan.

```bash
PORT      STATE SERVICE     REASON          VERSION
135/tcp   open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open              syn-ack ttl 127 Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49155/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49156/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49157/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery:
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-06-26T00:26:53+01:00
| smb2-security-mode:
|   2:1:0:
|_    Message signing enabled but not required
|_clock-skew: mean: -19m58s, deviation: 34m37s, median: 0s
| smb2-time:
|   date: 2023-06-25T23:26:57
|_  start_date: 2023-06-25T23:12:05
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 12383/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 28365/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 19006/udp): CLEAN (Timeout)
|   Check 4 (port 26942/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

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
