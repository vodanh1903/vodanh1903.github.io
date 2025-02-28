# Webcrypto

## Background

I think we can all agree that most of us grew up watching the iconic cartoon Tom & Jerry. Every kid would feel that surge of adrenaline during the thrilling chases and chaotic conflicts between the mischievous mouse and the ever-determined cat. The excitement of those scenes—the heart-pounding moments of escape—sometimes felt almost real.<br>

But then, I heard a little rumor: what if all those chases were fake? What if Tom and Jerry were actually friends all along? That revelation shook me. I had no one to ask about this mind-bending twist, so I decided to take matters into my own hands—I created a web app to settle this question once and for all.<br>

I know the truth now. Do you think you can uncover it too?<br>

[https://chal.acectf.tech/Webrypto/]()

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-24.jpg)

- **Difficulty: Easy**
- **This challenge doesn't have source code**

## Enumeration

Index page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-25.jpg)

The page shows us the source code of this web page. And we need to bypass the checking to get the flag. The page takes two `GET` parameters: `tom` and `jerry`, then it will compares the value of these two parameters with `!=` ( this is the **loose comparison** ). If we meet the conditions, it will continue to compare the MD5 hash of `ACECTF . $tom` with `ACECTF . $jerry` ( in [PHP](https://www.php.net/manual/en/language.operators.string.php), the `.` is the string concatenation operator), and if we meet the conditions again, we will get the flag!<br>

## Exploitation

This web page's checking method is likely to have a `Type Juggling` vulnerability because of the `!=` operator. We call `!=` and `==` the **loose comparison** because it only compares **the value**, not **the type** of the data, you can read more about it [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling).<br>

So if we send `GET` request with `tom[]=0&jerry[]=1`, the page will only compare the value of these two parameters, so we can bypass the first checking, and because we send two **empty arrays**, the MD5 hash of both parameters will return the same hash, and we can get the flag!

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/ACECTF-1.0/images/image-26.jpg)

> I think this challenge can be solved through hash collision, but I don't really know about it so...

- **Flag: `ACECTF{70m_4nd_j3rry_4r3_4ll135}`**

## Conclusion

What we've learned:

1. Type Juggling