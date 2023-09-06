---

title: "Jerry"
date: 2023-08-10T11:30:03+00:00
tags: ["web","windows","Apache Tomcat/Coyote JSP engine 1.1"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Jerry.png"
comments: false
description: "This is a friendly windows machine that is vulnerable because a low version of Apache Tomcat/Coyote JSP engine 1.1."

---

## Room Information

- Room name: Jerry
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Jerry

![Untitled](/HTB/Jerry.png)

## Tools Used

- Nmap
- wfuzz
- gobuster
- msfconsole

## Port Scanning

Other room, other port scan.

![Untitled](/HTB/nmap-jerry.png)

So let’s focus on port 8080 (Apache Tomcat/Coyote JSP engine 1.1)…

## Enumeration

As I always do with all machines I set out to do the directory scan with **gobuster** and **wfuzz,** and I find some things:

![Untitled](/HTB/wfuzz-jerry.png)

## Exploiting

The truth is that I spent quite a long time seeing if I could get something out of this and the website, but the solution was much easier than I thought.

After being a while with this, in a perhaps a little desperate way, I searched the whole name of the port 8080 version in google and I found a gem.

The website → https://charlesreid1.com/wiki/Metasploitable/Apache/Tomcat_and_Coyote

Then we just had to follow the steps, first we found out the directories of the web page, then the password of the `/manager/` and finally we uploaded the reverse shell and we entered as **root**, and all thanks to msfconsole.

Here are some of the screenshots I took during the process, but the web page I have left explains everything in much more detail:

```bash
use auxiliary/scanner/http/tomcat_mgr_login
```

![Untitled](/HTB/exploit1-jerry.png)

```bash
use exploit/multi/http/tomcat_mgr_upload
```

![Untitled](/HTB/exploit2-jerry.png)

Here is a video of the final part of how I accessed the machine as **root**.

![Untitled](/HTB/pwn-jerry.gif)


![Untitled](/HTB/root-jerry.png)

## Escalating Privileges

No privilege escalation was necessary on this machine 😇

## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about port 8080 and Apache Tomcat/Coyote JSP engine 1.1
    
- Where there any novel tools or techniques employed?
    
    No.
