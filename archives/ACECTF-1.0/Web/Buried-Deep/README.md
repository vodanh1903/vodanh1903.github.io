# Buried Deep

## Background

"I’m not a hacker. I’m just someone who wants to make the world a little better. But the world isn’t going to change itself."<br>

Submit your answer in the following format: ACECTF{3x4mpl3_fl4g}<br>

The flag content should be in lowercase letters only.<br>

[http://34.131.133.224:9998/]()

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-4.jpg)

- **Difficulty: Easy**
- **This challenge doesn't have source code**

## Enumeration

Index page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-5.jpg)

Nothing special, so I decides to check the source page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-6.jpg)

It has a link to the `style.css` file, I will check this file:

In this `style.css` file, there is a tag named `#flag`. This `#flag` tag has a `content` sounds suspicious: ``bC5 !2CE @7 E96 u=28 :D i f9b0db4CbEd0cCb03FC`b5N``. I think the `content` has been rotated, I will use [dcode.fr](https://www.dcode.fr/rot-cipher) to rotate it:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-7.jpg)

We find the 3rd part of the flag: **`7h3_53cr3t5_4r3_bur13d}`**, lol.<br>

So the first and the second part are still somewhere on the web. Now, I will check if the page has the `robots.txt` file:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-8.jpg)

And yes! It has the `robots.txt` file with many hidden files:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-9.jpg)

After checking all the hidden files, I find two files interesting: `buried` and `secret path`.<br>

Content of `buried` file:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-10.jpg)

This is the `ASCII` format, I will use [duplichecker](https://www.duplichecker.com/ascii-to-text.php) to convert from `ASCII` decimal to text:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-11.jpg)

First part of the flag: **`ACECTF{1nf1l7r471ng_7h3_5y573m_`**

Content of `secret path` file:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-12.jpg)

This is the `morse` code. I will use [dcode](https://www.dcode.fr/morse-code) to translate this `morse` code:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-13.jpg)

We got the second part of the flag: **`15_345Y_WH3N_Y0U_KN0W_WH3R3_`**. Since the flag format is in lowercase character, I will change the 2nd part to **`15_345y_wh3n_y0u_kn0w_wh3r3_`**.

## Exploitation

Combines three parts and we will get the complete flag:

- **Flag: `ACECTF{1nf1l7r471ng_7h3_5y573m_15_345y_wh3n_y0u_kn0w_wh3r3_7h3_53cr3t5_4r3_bur13d}`**

## Conclusion

What we've learned:

1. Reconnaissance and knowledge about some encode: ASCII, morse.