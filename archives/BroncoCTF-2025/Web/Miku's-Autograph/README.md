# Miku's Autograph

## Background

I am so proud of the fact that I have Miku's autograph. Ha! You don't!

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-7.png)

- **Difficulty: Medium** (I think it's an easy challenge, but it's really hard for me)
- **This challenge doesn't have source code**

## Enumeration

Access the link, it leads me to the page that has a `Magic Miku Login` button. When I click on it, it returns a text: `Access Denied: You are not miku_admin!`.

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-8.png)

Maybe this challenge is related to cookies, so I decides to check the cookies of this page, but nothing shows up:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-9.png)

Hmm, maybe I should check on the source of this page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-10.png)

The page has a script that when I click on the `Magic Miku Login` button, it will `fetch` the content of the endpoint `/get_token`, then passes the data to `magic_token` variable and sends it to the endpoint `/login`. Finally, it will return the result to an `innerHTML`.<br>

First, I will check on the endpoint `/get_token` to see what's in there:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-11.png)

This endpoint contains JSON data which has a `your_token` key with a `JWT` format value, the JWT changes whenever I reload the page.

> A `JSON web token (JWT)` is mainly composed of `header`, `payload`, `signature`. These three parts are separated by `dots` ( . ). You can read more about it [here](https://jwt.io/introduction)

I will use [jwt.io](https://jwt.io/) to parse this JWT:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-12.png)

This is a JWT with `HS256` algorithm, and with the value of `sub` in the payload, we know that we are currently a `miku_user`. First, I will change the value of `sub` to `miku_admin` to see if I can get the flag with this new JWT ( Although we need to know the secret key so we can have a verify signature when changing a value, maybe the server won't check the signature so we can get the flag ).

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-13.png)

I will use curl to `POST` data to the endpoint `/login`:

```bash
curl -X POST -d "magic_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtaWt1X2FkbWluIiwiZXhwIjoxNzM5ODYyNDQxfQ.7JyvQg5m83bZYb2LGu9AhTvUovWcEEZkuuQqVFbtUfU" https://miku.web.broncoctf.xyz/login
```

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-14.png)

It says `Invalid Magic Login!`. Hmm, not easy like that, I guess? Next, I decides to remove the signature part of the JWT to see if I can bypass the server ( Remember, when removes the signature part, you shouldn't remove the dot because the dot maybe tricks the server that we send to it a JWT with full three parts ):

```bash
curl -X POST -d "magic_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtaWt1X2FkbWluIiwiZXhwIjoxNzM5ODYyNDQxfQ." https://miku.web.broncoctf.xyz/login
```

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-15.png)

It still returns `Invalid Magic Login!`. Okay, now I will perform dictionary attack to find the secret key, I will use `hashcat` with the wordlist `rockyou` to find the secret key.

```bash
hashcat -a 0 -m 16500 "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtaWt1X3VzZXIiLCJleHAiOjE3Mzk4NjQwNzl9.ydrqrbtADQ-0pzttleUQdXQFYlL1HNEkhkJKyz2gXkU" /usr/share/wordlists/rockyou.txt
```

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-16.png)

And `hashcat` returns the `exhausted` status, which means no matching secret key was found. This is the most desperate moment for me, I spent hours and hours with different tools, different wordlists just to find the secret key, but nothing works. ( I'm so noob lol, partly because I don't have much experience with JWT ).<br>

At last, I found a tool called [jwt_tool](https://github.com/ticarpi/jwt_tool). Reading through its documentation, this tool have an exploitation mode with `-X` switch. This mode has an attack method called `a` that is to set the value of `alg` in the header to `none` then removes the signature part.<br>

Hmm, this sounds interesting, I will use the default JWT of the `/get_token` endpoint to test the exploit because when we send the default token to the `/login` endpoint, we will receive the `Access Denied: You are not miku_admin!` message, and if the exploit successes, it will also return the `Access Denied: You are not miku_admin!` message otherwise it will return the `Invalid Magic Login!` message.

```bash
python jwt_tool.py "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtaWt1X3VzZXIiLCJleHAiOjE3Mzk4NjgxMTJ9.38GFWNGYSf6LAwEVWW-xeWUNzD98o0GHs7CLTtkirf0" -X a
```

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-17.png)

We will `POST` this JWT to the `/login` endpoint to see the result:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-18.png)

And yesss!! This is the result that we want!

## Exploitation

Now, using this attack method, we need to change the value of `sub` to `miku_admin`, and we can also use this tool to do that with `-I` to update existing claims with new values, `-pc` switch defines the `payload` and `-pv` switch defines the new value that we want to change.

```bash
python jwt_tool.py "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtaWt1X3VzZXIiLCJleHAiOjE3Mzk4NjgxMTJ9.38GFWNGYSf6LAwEVWW-xeWUNzD98o0GHs7CLTtkirf0" -X a -I -pc sub -pv miku_admin
```

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-19.png)

Let's see if we can get the flag with this exploit:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-20.png)

Finally, I feel so happy when I get the flag of this challenge, OMG!

- **Flag: `bronco{miku_miku_beaaaaaaaaaaaaaaaaaam!}`**

## Conclusion

What we've learned:

1. JWT authentication bypass via alg header injection ( **we can set the alg header to none value** ).