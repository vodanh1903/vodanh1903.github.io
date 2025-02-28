# Bucket List

## Background

You know what's a bucketlist? In simple terms, it's just a list of wishes people want to achieve before the leavee this world. I found it to be very limiting & ironic because how can you know when you'll leave the world behind? It's better to enjoy every moment and take on every opportunity you can. One of my whishes though is to pet a cat, do you mind checking this one out. So cute.<br><br>

[What a cutie patootie!]()

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image.jpg)

- **Difficulty: Easy**
- **This challenge doesn't have source code**

## Enumeration

Click on the link given by the challenge, we are redirected to a page has a picture of a cat:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-1.jpg)

After reconnaissance, the picture is nothing special. Looking at the URL, we are currently at the path `/fun/can_we_get_some_dogs/026.jpeg`. So I decides to check the path `/` to see if something interesting:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-2.jpg)

It leads me to the page contains a `XML` file with many `Contents`. After scrolling through this `XML` file, I finds the `Contents` with the `Key` sounds interesting: `<Key>cry-for-me/acectf/secret.txt</Key>`. So I decides to check this path:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-3.jpg)

## Exploitation

This page has a string seems like a base64-encode string. Decoding the string and we will get the flag:

```bash
┌──(vodanh㉿kali)-[~]
└─$ echo "QUNFQ1RGezdoM180dzVfMTVfbTE1YzBuZjE2dXIzZH0=" | base64 -d
ACECTF{7h3_4w5_15_m15c0nf16ur3d}
```

- **Flag: `ACECTF{7h3_4w5_15_m15c0nf16ur3d}`**

## Conclusion

What we've learned:

1. Reconnaissance