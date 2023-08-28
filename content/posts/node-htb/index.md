---
title: "Node"
date: 2023-08-10T11:30:03+00:00
tags: ["web","linux","mongodb","networks"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Node.png"
comments: false
description: "This is a friendly linux machine that has a web page vulnerable due to bad configuration of some directories."
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

- Room name: Node
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Node

![Untitled](/HTB/Node.png)


## Tools Used

- Nmap
- searchsploit
- 7z
- wfuzz
- unzip
- base64
- gobuster
- burpsuite
- whatweb
- john
- zip2john

## Port Scanning

Other room, other port scan.

![Untitled](/HTB/nmap-node.png)

I found 2 open ports, 22(ssh), 3000(web). Let's focus on the website throw port 3000...

## Enumeration

On the directory listing with `gobuster` and `wfuzz` It throws me nothing relevant so I used other methods.

![Untitled](/HTB/wfuzz-node.png)

This is more or less the view’s of the home page of node.

![Untitled](/HTB/backup-node.png)

With the terminal of firefox I saw a page in networks, so sospicious :)

![Untitled](/HTB/probando-node.png)

## Exploiting

So I visit the following url `http://10.10.10.58:3000/api/users/` and I saw some credentials encoded in SHA256.

![Untitled](/HTB/hash-node.png)

So let's try to decode them with:

```bash
 john --wordlist=/usr/share/wordlists/rockyu.txt hash --format=Raw-SHA256
```

![Untitled](/HTB/john-node.png)

![Untitled](/HTB/2-node.png)

But with this credentials I couldn’t do anything, because I wasn’t admin.

Then I tried to change the url and I found other page with admin credentials too :))

![Untitled](/HTB/admin-node.png)

So when I decode the admin credential and tried to login to the page it throws me a download of a **Backup,** but the download failed a lot and I couldn’t download the backup, so I do it with burpsuite

![Untitled](/HTB/donwl-node.png)

and then I copy the results, which was encoded on Base64, and decode it on my terminal.

![Untitled](/HTB/file1-node.png)

```bash
cat myplace.backup | base64 -d | sponge myplace.backup
```

![Untitled](/HTB/tofile2-node.png)

![Untitled](/HTB/file2-node.png)

So it results that was a Zip file …

Then I did a `7z l myplace.backup` to list al files of the zip.

![Untitled](/HTB/7zl.png)

I then used `zip2john myplace.backup > hashzip` to create a hash so I could decrypt it with  `john --wordlist=/usr/share/wordlists/rockyu.txt hash --format=Raw-SHA256`.

![Untitled](/HTB/zipjohn.png)

After that I downloaded it and took a look at it. Almost the first file I saw was one called **app.js** and I saw that it contained some credentials of a user named mark :)

![Untitled](/HTB/mark-node.png)

Also I saw that this url tries to connect to **mongodb,** so suspicious …

Next I could connect by ssh on user mark…

![Untitled](/HTB/markshell-node.png)

## Treatment of tty

```bash
 script /dev/null -c bash
 ctrl + z
 stty raw -echo; fg
 restet xterm
 export TERM=xterm
```

## Escalating Privileges

Next I run the usual things that I always do:

```bash
 sudo -l
 cat /etc/crontab
 find / -perm -4000 2>/dev/null
 find / -type f -writable 2>/dev/null
 bash procmon.sh
 lsb_release -a
 uname -a
 id
 getcap -r / 2>/dev/null
```

What I could see in the analysis I have run is that there is a process that runs the user **Tom** called **backup** which can execute functions in **root** mode, so it only remains to become **Tom** :)

While doing this I remembered that where I got the credentials to log in as **mark** there was some kind of connection to the database using **mongodb**, so I'm going to try to connect to this one and try to execute commands by Tom (i take a look of some files like app.js to could do this)...

![Untitled](/HTB/tom-node.png)

So now I was user Tom 😎

Next I start searching how worked backup file and I found some things…

```bash
tom@node:/usr/local/bin$ ltrace backup 
__libc_start_main(0x80489fd, 1, 0xff9bc6d4, 0x80492c0 <unfinished ...>
geteuid()                                        = 1000
setuid(1000)                                     = 0
exit(1 <no return ...>
+++ exited (status 1) +++
```

Doing some tests I realized that this program uses 3 arguments, the first one is always -q, the second one is always the same key since it checks it later, and then the third argument is the directory which we want to backup ... interesting 😇

It didn't take me long to realize that I could backup the root folder, then unzip it and be able to log in without being root...(I found the key on app.js)

![Untitled](/HTB/backuproot-node.png)

When I did this, the backup file returned another base64 file (which I assumed would be a zip), then I took it to my main computer, decoded it and saw that it was a zip.

![Untitled](/HTB/juan-node.png)

![Untitled](/HTB/juanzip-node.png)

So I just hadto unzip it and do a cat of the flag 😎

## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about the firefox terminal, zip archives and Base64. 
    
- Were there any novel tools or techniques employed?
    
    The use base64, zip2john.
