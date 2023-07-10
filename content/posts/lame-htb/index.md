---
title: "Lame"
date: 2023-06-24T11:30:03+00:00
tags: ["smb","linux","tcpdump"]
categories: ["hackthebox"]
author: "Ro Hackss"
image: "/HTB/lame-icon.png"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This is a friendly Linux machine with a vulnerable Samba service that can be exploited."
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

- Room name: Lame
- Difficulty level: Easy
- Room link: [https://app.hackthebox.com/machines/Lame](https://app.hackthebox.com/machines/Lame)

![Untitled](/HTB/lame-icon.png)

## Tools Used

- Nmap
- Searchsploit
- smbclient
- crackmapexec
- tcpdump 


## Code Review & Exploitation

Upon running Nmap, I discovered multiple open ports, the three most interesting are: SSH(22), FTP(21), and Samba(445).

![Untitled](/HTB/lame-escaneo.png)

The FTP service is vsftpd 2.3.4, and my initial attempt was to connect anonymously. I managed to gain access, but it seemed limited. However, exploring through FTP did not yield much information.

![Untitled](/HTB/lame-2.png)

In Searchsploit, I found a potential exploit for vsftpd 2.3.4.

![Untitled](/HTB/lame-3.png)

Unfortunately, this exploit did not work effectively, so I decided to focus on the Samba service.
I personaly always do a scan with smbclient and crackmapexec, as follows:

```bash
smbclient -L 10.10.10.3 -N
crackmapexec smb 10.10.10.3
```

And it shows me some directories and information like if samba is signed (if isn't signed it's less secure). 

![Untitled](/HTB/lame1.png)

![Untitled](/HTB/lame3.png)

Then with searchsploit i do the same thing like bifore, but searching with a the version of samba:

![Untitled](/HTB/lame-4.png)

*OPTION 1*

I attempted the "username map script" exploit ([CVE-2007–2447](http://cvedetails.com/cve/cve-2007-2447)), which appeared to be the most promising way to authenticate.

![Untitled](/HTB/lame-5.png)

*OPTION 2*

On this case we learn more and i thing that this is the most important. So we start searching with 'batcat' this:

![Untitled](/HTB/descarga-exploit.png)

In this exploit we can see that we can gain access to de machine by using a username like that:

![Untitled](/HTB/exploit-analizado.png)

Also how at first i had discovered that i could connect by Samba using -N (anonymous user), so lets connect to the machine using to tmp for example:

```bash
smbclient //10.10.10.3/tmp -N

```

Once connected we will open the help menu, and we can see that it has logon program, so we can authenticte like a user using
what we have learned before, to test it I will try to send a ping and see if I get to receive it:

![Untitled](/HTB/logon.png)

![Untitled](/HTB/ping-recivido.png)

We can see that I received the ping, so we already known that we can do a reverse shell instead of a ping...


Finally, we gained access.

## Flag(s)

User’s flag

![Untitled](/HTB/lame-6.png)

Root’s flag

![Untitled](/HTB/lame-7.png)

## Lessons Learned

- What did you learn from the room?
    
    I learned how to exploit Samba and gained a basic understanding of machine attack methodology.
    
- Were there any new tools or techniques used?
    
    No new tools were used, but I improved upon my previous knowledge.
    
- What would you do differently next time?
    
    Next time, I would aim to capture better screenshots of my progress. 😅

**Thank you for reading, and happy hacking! 😄**
