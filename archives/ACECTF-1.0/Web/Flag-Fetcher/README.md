# Flag-Fetcher

## Background

Hey guys, I created a flag fetcher using some web stacks & technologies. It was supposed to fetch the flag.webp image file which contains the flag but there was some kind of error in doing that. Can you verify it? Maybe just get the flag I don't really care if you fix it or not.<br>

[This should've worked]()

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-20.jpg)

- **Difficulty: Easy**
- **This challenge doesn't have source code**

## Enumeration

Access the link, it will redirect us to a page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-21.jpg)

And this page keeps loading until it redirects to the `/flag.webp` path and an image appears:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-22.jpg)

## Exploitation

I have looked through the source page but nothing special. So I will use **Burp Suite** to capture all the request when we access this page to see if something interesting:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-23.jpg)

Access the HTTP history, and we will see that before it completely loads the image, it continuously sends request to some **path** with only one character. And if we notice, we will see that if we combines all these characters, we will get the flag.

> %7B and %7D are URL-encode of { and } respectively
> Instead of `Burp Suite`, you can use the `Network` tab of the `Developer tools` on the browser to capture all requests 

- **Flag: `ACECTF{r3d1r3ct10n}`**

## Conclusion

What we've learned:

1. Capture request with Burp Suite.