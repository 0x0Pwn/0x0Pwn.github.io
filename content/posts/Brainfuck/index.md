---
title: "BrainFuck"
date: 2023-06-24T11:30:03+00:00
tags: ["pop3","wordpress","linux"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Brainfuck.png"
comments: false
description: "This is a friendly Linux machine with a vulnerable Wordpress page web that can be exploited abusing his plugins."
---

Room Information

- Room name: Brainfuck
- Difficulty level: Insane
- Room link: [https://app.hackthebox.com/machines/Brainfuck](https://app.hackthebox.com/machines/Brainfuck)

![Untitled](/HTB/Brainfuck.png)

## Tools Used

- Nmap
- Searchsploit
- nc
- wpscan
- wfuzz
- openssl
- sslscan 
- john
- ssh2john
- ssh

## Code Review & Exploitation

Upon running Nmap, I discovered multiple open ports, the most interesting are: SSH(22), smtp(25), 110(email pop3)
, 443(https).

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.17 -oN escaneo
sudo nmap -sC -sV -p22,25,110,143,443 10.10.10.17 -oN targeted
```
![Untitled](/HTB/targeted-brainfuck.png)

The SSH service is 7.2p2, and my initial attempt was to search with searchsploit if it has some vulnerabilities.
And I find one that allows to enumerate users (orestis ...).

Then I put the ip's on my /etc/hosts and search for information about them. First of all I run `whatweb https://10.10.10.17/`,
and `openssl s_client -connect 10.10.10.17:443` (to list information and posible users). This throws me that:

![Untitled](/HTB/orestis-brainfuck.png)

One thing so interesting is that I found a user called **orestis** ...
Then I run a `sslscan https://10.10.10.17/` to list more information.

![Untitled](/HTB/sslscan-brainfuck.png)

After that I run a what web with the first DNS name `whatweb https://brainfuck.htb/`

![Untitled](/HTB/wordpres-brainfuck.png)

and it throws me that the web is running on a wordpres server :). So let's do a wpscan, 
`wpscan --url https://brainfuck.htb --enumerate u,p --disable-tls-checks` to enumerate all
users and plugins (are the most susceptible to have vulnerabilities).

![Untitled](/HTB/wpscan-brainfuck.png)

Some thing so interesting is that it throws me that there's a plugin called **wp-support-plus-responsive-ticket-system**,
and the version is out of date ...Immediately I search it with searchsploit and surprise, it's vulnerable.
I download de script and read it to understand how it works

![Untitled](/HTB/searchsploit-brainfuck.png)

It tell's that I can log without any password ...

Then I put the proof of concept on a html file and run a http server with python...

![Untitled](/HTB/payload-brainfuck.png)

Before of this plugin abuse I already have done a gobuster scan that throws me that we have directory listing,
and with that I know that there's a login panel.

```bash
gobuster dir -u https://brainfuck.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -k
```

![Untitled](/HTB/wps-content-brainfuck.png)

So after running the above script, I'm now admin user. Now I'm going to take a look of the theme menu if I can modify
some pages to introduce a php reverse shell, but I didn't have write permision. Then i search for the plugins, and
on email plugin I found a password. This is so interesting because we already known that 110 port is open, so let's try to connect with netcat.

```bash
nc 10.10.10.17 110
```

![Untitled](/HTB/nc-brainfuck.png)

This services allowed me to connect with the above password I found. So let's list all email's.
In one email I found another password for user orestis, and some thing interesting that said's on
the email is that this user/password is for a secret forum :), instanntly I thought on the other DNS
**sup3rs3cr3t.brainfuck.htb**

![Untitled](/HTB/secret-brainfuck.png)

So I searched for the page web and log with orestis/(passwd), instantly I realized that the forum discusion
was between the admin and orestis. One of the chats (the most interesting) seems to be cyphred, so I search 
which were the most used cypher methods and probe with their online traducer. 

After a while I found one called vigenere that worked with a keyword. I soon deduced that if I found a word 
that I knew how to translate and put the translation in the key I would return the real key repeated along the 
chain. So I took advantage of an Orestis mistake since he always signed his messages and I got the key and 
translated the entire chat until I found a route that pointed to a id_rsa, that of orestis. 

![Untitled](/HTB/vigenere-brainfuck.png)

You remember the id_rsa that I downloaded thanks to the above, don't you? Well, when I try to connect throw 
ssh it asks me for a password. So let's convert the id_rsa to a hash with `python2 ~/programas/ssh2john.py id_rsa`
and try to decrypt it with john the ripper.

![Untitled](/HTB/ssh2jhon-brainfuck.png)

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
![Untitled](/HTB/psw-brainfuck.png)

So the password is **3poulakia!**

![Untitled](/HTB/pwnd-brainfuck.png)

Finally, we gained access.

## Escalating privileges

One of the first things I allways do is looking for the id of the groups, and sourprise orestis has **lxd**.
We all know that lxd/docker are vulnerable to a escalate of privileges.

![Untitled](/HTB/lxd-docker-brainfuck.png)

So let's find a script with searchsploit and pawn it.

![Untitled](/HTB/id-brainfuck.png)

I followed the steps in the script, I downloaded the files it asked for, I deleted one thing that gave problems,
then I transferred them to the victim pc thanks to the fact that I created a server on port 80 that pointed to 
the folder where I had everything downloaded.

![Untitled](/HTB/wget-brainfuck.png)

So let's execute the script ...

![Untitled](/HTB/exploit-brainfuck.png)

Now we are root !!!

## Lessons Learned

- What did you learn from the room?
    
    I learned how to exploit a wordpress web understanding of machine attack methodology.
    
- Were there any new tools or techniques used?
    
    I used wpscan, ssh2john, john, sslscan ...
    
- What would you do differently next time?
    
    Next time, I would aim to capture better screenshots of my progress. 😅

**Thank you for reading, and happy hacking! 😄**
