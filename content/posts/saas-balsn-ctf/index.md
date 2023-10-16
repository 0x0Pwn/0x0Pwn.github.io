---

title: "SaaS (balsn)"
date: 2023-08-10T11:30:03+00:00
tags: ["web","fast-json-stringify-compiler"]
categories: ["CTF’s"]
author: "0x0Pwn"
image: "/CTFs/1-saas.png"
comments: false
description: "This machine presents a **backend** server which we can breach thanks to a remote execution of commands in the POST request due to the **fast-json-stringify-compiler** library."

---

## Room Information

- Room name: SaaS
- Difficulty level: unknown (easy I think)
- Room link: https://balsnctf.com/challenges

![Untitled](/CTFs/1-saas.png)

## Tools Used

- Docker
- Visual Studio
- Research

## Enumeration

If we make a request to the url / default of the website, we get the error 404 response.

![Untitled](/CTFs/2-saas.png)

![Untitled](/CTFs/3-saas.png)

But if instead, we change as it says in `default.conf`, the host to "easy++++++" and the **url** to http://a.saas/, then we get a connection.

![Untitled](/CTFs/4-saas.png)

## Explotation

So based on this, I started to investigate the index.js file where I found two different paths where we can possibly find the vulnerability:

- The first one `“/register”` is a post type request, in which according to the code we can send any kind of message in `json`, since the input format is not configured, so here we can try to test if there is any `command injection` in `js` or similar.
    
    ![Untitled](/CTFs/5-saas.png)

    So what I tried was the following: after trying several ways to get remote command execution by sending the malicious json, I realized that I was not going to find the key, so I started searching like crazy on the internet and found these pages…
    
    [fast-json-stringify - npm (npmjs.com)](https://www.npmjs.com/package/fast-json-stringify)
    
    [Node.js third-party modules | Report #532667 - Server Side JavaScript Code Injection | HackerOne](https://hackerone.com/reports/532667)
    
    In which I realised that thanks to the `required` method of `fast-json-stringify-compiler`, i was able to execute commands remotely and get the flag file listed…
    
    Obviously it could not list it directly, but the post method used the malicious schema that we provided and a random number (`uuid`) to create a validation, which I later saw that it used in the other route to check if the `uuid` was validated.
    
    ![Untitled](/CTFs/6-saas.png)
    

- The second was `"/whowilldothis/:uid"`, as you can see below:
    
    ![Untitled](/CTFs/7-saas.png)
    
    So in this route we add the uuid returned by the previous request and get the flag…👹

    ![Untitled](/CTFs/8-saas.png)




