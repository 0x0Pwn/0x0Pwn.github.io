---
title: "Nibbles"
date: 2023-07-04T13:02:46+02:00
tags: ["web","privesc","linux"]
categories: ["hackthebox"]
author: "Ro Hackss"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This room is a Linux environment running a website powered by the Nibbleblog engine, and the version we are using has a shell vulnerability."
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

- Room name: Nibbles
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Nibbles

![Untitled](/HTB/nibbles-icon.png)

## Tools Used

- Nmap
- Gobuster

## Port Scanning

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD8ArTOHWzqhwcyAZWc2CmxfLmVVTwfLZf0zhCBREGCpS2WC3NhAKQ2zefCHCU8XTC8hY9ta5ocU+p7S52OGHlaG7HuA5Xlnihl1INNsMX7gpNcfQEYnyby+hjHWPLo4++fAyO/lB8NammyA13MzvJy8pxvB9gmCJhVPaFzG5yX6Ly8OIsvVDk+qVa5eLCIua1E7WGACUlmkEGljDvzOaBdogMQZ8TGBTqNZbShnFH1WsUxBtJNRtYfeeGjztKTQqqj4WD5atU8dqV/iwmTylpE7wdHZ+38ckuYL9dmUPLh4Li2ZgdY6XniVOBGthY5a2uJ2OFp2xe1WS9KvbYjJ/tH
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPiFJd2F35NPKIQxKMHrgPzVzoNHOJtTtM+zlwVfxzvcXPFFuQrOL7X6Mi9YQF9QRVJpwtmV9KAtWltmk3qm4oc=
|   256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC/RjKhT/2YPlCgFQLx+gOXhC6W3A3raTzjlXQMT8Msk
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The results shows that we have two open ports one running http service at port 80 and other and other running a ssh service at port 22.

## Enumeration

Let’s focus on the website.

![Untitled](/HTB/nibbles-1.png)

The website shows a Hello World message only.

![Untitled](/HTB/nibbles-2.png)

If we go to the source code we can see that there is a commented line that told us where is the real website running.

![Untitled](/HTB/nibbles-3.png)

With gobuster I am going to fuzz the website.

```bash
gobuster dir --url http://10.10.10.75/nibbleblog -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,js,css,py,html,cgi,sh,txt
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.75/nibbleblog
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /Users/aymanboulaichdahhou/Documents/wordlists/KaliLists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              css,py,html,cgi,sh,txt,php,js
[+] Timeout:                 10s
===============================================================
2023/07/04 00:02:39 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 301]
/.html                (Status: 403) [Size: 302]
/.hta                 (Status: 403) [Size: 301]
/.hta.txt             (Status: 403) [Size: 305]
/.hta.php             (Status: 403) [Size: 305]
/.hta.js              (Status: 403) [Size: 304]
/.hta.css             (Status: 403) [Size: 305]
/.hta.html            (Status: 403) [Size: 306]
/.hta.py              (Status: 403) [Size: 304]
/.hta.cgi             (Status: 403) [Size: 305]
/.htaccess.html       (Status: 403) [Size: 311]
/.htaccess.cgi        (Status: 403) [Size: 310]
/.hta.sh              (Status: 403) [Size: 304]
/.htaccess.sh         (Status: 403) [Size: 309]
/.htaccess            (Status: 403) [Size: 306]
/.htaccess.php        (Status: 403) [Size: 310]
/.htaccess.txt        (Status: 403) [Size: 310]
/.htaccess.js         (Status: 403) [Size: 309]
/.htaccess.css        (Status: 403) [Size: 310]
/.htaccess.py         (Status: 403) [Size: 309]
/.htpasswd            (Status: 403) [Size: 306]
/.htpasswd.cgi        (Status: 403) [Size: 310]
/.htpasswd.py         (Status: 403) [Size: 309]
/.htpasswd.html       (Status: 403) [Size: 311]
/.htpasswd.sh         (Status: 403) [Size: 309]
/.htpasswd.txt        (Status: 403) [Size: 310]
/.htpasswd.php        (Status: 403) [Size: 310]
/.htpasswd.js         (Status: 403) [Size: 309]
/.htpasswd.css        (Status: 403) [Size: 310]
/admin                (Status: 301) [Size: 321] [--> http://10.10.10.75/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/admin.php            (Status: 200) [Size: 1401]
/content              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/content/]
/feed.php             (Status: 200) [Size: 302]
/index.php            (Status: 200) [Size: 2987]
/index.php            (Status: 200) [Size: 2987]
/install.php          (Status: 200) [Size: 78]
/languages            (Status: 301) [Size: 325] [--> http://10.10.10.75/nibbleblog/languages/]
/LICENSE.txt          (Status: 200) [Size: 35148]
/plugins              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/plugins/]
/README               (Status: 200) [Size: 4628]
/sitemap.php          (Status: 200) [Size: 401]
/themes               (Status: 301) [Size: 322] [--> http://10.10.10.75/nibbleblog/themes/]
/update.php           (Status: 200) [Size: 1622]
```

We can find some useful information about the software of the website in the README file.

![Untitled](/HTB/nibbles-4.png)

We can see that is performing nibbleblog which is a blogging engine powered by PHP and the version that is using is the v4.0.3.

![Untitled](/HTB/nibbles-5.png)

## Exploiting

Let’s see if there are some vulns for that version.

![Untitled](/HTB/nibbles-6.png)

In google I found one [article](https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html) that don’t mention metasploit.

This article explain how works a shell vuln that helps you to do RCE with a reverse shell. The article explain that if we have credentials we can take advantage of the my image plugin preinstalled in all the nibbleblogs with our version to upload a php reverse shell.

Now our problem is the credentials if we navigate to  http://10.10.10.75/nibbleblog/admin.php we need the credentials to manage the website.

![Untitled](/HTB/nibbles-7.png)

I tried the default passwords first admin/- -/admin admin/nibbles nibbles/admin nibbles/- -/nibbles.

I hace managed to enter with admin/nibbles and without bruteforcing the website.

We need now to navigate to Plugins > Mi image > Config as says the article.

![Untitled](/HTB/nibbles-8.png)

The vuln’s article says that we need to put the reverse shell as a image and ignore the warnings I uploaded this [reverse shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).

Let’s set up a listener in the port 4444

```bash
nc -lv 4444
```

Once done we need to move to http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php

## Gaining an Initial Foothold

We are in now with user privileges. 

![Untitled](/HTB/nibbles-9.png)

**User’s flag**

![Untitled](/HTB/nibbles-10.png)

## Escalating Privilege

If we use sudo -l we observe a .sh that can be ran by sudo without password this .sh is found at /home/nibbler/personal/stuff/monitor.sh

![Untitled](/HTB/nibbles-11.png)

If we unzip the zip we can find this .sh and modifying his content to “/bin/bash” we can run a shell with root privilege thanks to the sudo command.

```bash
sudo /home/nibbler/personal/stuff/monitor.sh
```

Let’s check if we are root users now.

![Untitled](/HTB/nibbles-12.png)

**Root’s flag**

![Untitled](/HTB/nibbles-13.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I learned the importance of changing the default credentials if this thing was changed maybe the exploitation of the machine gonna be more harder.
    
- Were there any novel tools or techniques employed?
    
    Other way to exploit the sudo -l by modifying the content of a .sh which let us have privesc.

**Thank you for reading, and happy hacking! 😄**
