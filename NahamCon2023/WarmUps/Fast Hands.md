>Author: @JohnHammond#6971  
>
>You can capture the flag, but you gotta be fast!
--------------------------------------
We are given a button to Capture The Flag:

![Pasted image 20230616141050.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Warmup/Images/Pasted%20image%2020230616141050.png)

Clicking the button opens a new window that closes rather abruptly. The challenge prompt suggests we be quick, so I am thinking a `curl` request might be good since the contents do not close. Right clicking does not, however, allow me to copy a link url. 
I decided to see how this action is behaving when redirected through burp:

![Pasted image 20230616141445.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Warmup/Images/Pasted%20image%2020230616141445.png)
Interception does indeed work great. We can see on click, we are redirected to /capture_the_flag.html.

When we manually follow the link:

![Pasted image 20230616141617.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Warmup/Images/Pasted%20image%2020230616141617.png)

It's another button, but doesn't appear to be working too well. Viewing page source with Ctrl+U, we can see the flag within the code:
```html
<div class="container p-5">
            <div class="text-center mt-5 p-5">
                <button type="button" onclick="ctf()" class="btn btn-success"><h1>Your flag is:<br>
                  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  <span style="display:none">
                  flag{80176cdf1547a9be54862df3568966b8}
                </span></button>
            </div>
```

`flag{80176cdf1547a9be54862df3568966b8}`


Alternatively, we could have curled http://challenge.nahamcon.com:31565/capture_the_flag.html to achieve the same results.

```bash
$ curl -s http://challenge.nahamcon.com:31565/capture_the_flag.html | grep flag

                <button type="button" onclick="ctf()" class="btn btn-success"><h1>Your flag is:<br>
                  flag{80176cdf1547a9be54862df3568966b8}
```

