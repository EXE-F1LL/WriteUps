>Author: @JohnHammond#6971  
>  
>Look at this fan page I made for the Hidden Figures movie and website! Not everything is what it seems!
----------------------------------------------
Entering the website we see a pretty well-constructed page promoting `Hidden Figures`, a pretty popular book/movie. None of the links take us to other pages, so there is little to be explored.
Early on, I noticed that folder directory `/assets/` is free to view. I thought perhaps there might be something hidden here, or in another commonly used folder type? 

![Pasted image 20230616185356.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616185356.png)

After some lengthy enumeration, I found nothing of interest.

Looking at the page source, I noticed that there are several links with *very* long url codes:
```
data:image/png;base64,/9j/4AAQSkZJRgABAQEBLAEsAAD/7RqcUGhvdG9zaG9wIDMuMAA4QklNBAQAAAAAAFocAVoAAxslRxwCAAACAAAcAngAFVdvbWFuIG9mIE5BU0EgTGFuZ2xleRwCVQAPRGF2aWQgQy4gQm93bWFuHAI3AAgyMDE0MDMyMBwCPAALMDkxNTA2KzAwMDA4QklNBCUAAAAAABBbRWm4H1TKYVo1ObYsy6/POEJJTQQ6AAAAAAESAAAAEAAAAAEAAAAAAAtwcmludE91dHB1dAAAAAUAAAAAUHN0U2Jvb2wBAAAAAEludGVlbnVtAAAAAEludGUAAAAASW1nIAAAAA9wcmludFNpeHRlZW5CaXRib29sAAAAAAtwcmludGVyTmFtZVRFWFQAAAAWAEUAUABTAE8ATgAgAFMAdAB5AGwAdQBzACAAUAByAG8AIAA3ADkAMAAwAAAAAAAPcHJpbnRQcm9vZlNldHVwT2JqYwAAAAwAUAByAG8AbwBmACAAUwBlAHQAdQBwAAAAAAAKcHJvb2ZTZXR1cAAAAAEAAAAAQmx0bmVudW0AAAAMYnVpbHRpblByb29mAAAADHByb29mTW9uaXRvcjhCSU0EOwAAAAACLQAAABAAAAABAAAAAAAScHJpbnRPdXRwdXRPcHRpb25zAAAAFwAAAABDcHRuYm9vbAAAAAAAQ2xicmJvb2wAAAAAAFJnc01ib29sAAAAAABDcm5DYm9vbAAAAAAAQ250Q2Jvb2wAAAAAAExibHNib29sAAAAAABOZ3R2Ym9vbAAAAAAARW1sRGJvb2wAAAAAAEludHJib29sAAAAAABCY2tnT2JqYwAAAAEAAAAAAABSR0JDAAAAAwAAAABSZCAgZG91YkBv4AAAAAAAAAAAAEdybiBkb3ViQG/gAAAAAAAAAAAAQmwgIGRvdWJAb+AAAAAAAAAAAA
<...SNIP...>
```
And so on. I didn't know this was possible or ever really done, but it appears that some websites might provide their images in base64 encoded data like this so that images can be loaded without retrieving data from other pages. This seems to be what is going on here, as following the link takes me to an image regardless of the website's running state.

Eventually I thought to download each of the images located on the website. While reviewing this challenge I discovered that saving the image directly is enough, but at the time I had decided to copy/paste the entire url into a file. After removing `data:image/png;base64,` I base64 decoded the info:

```
$ cat url | base64 -d > decode_url 
```

And so on, for all 4 images found.

Checking `exiftool` didn't give me any critical information, however `binwalk` did show me something *hidden* in the figures.

```bash
$ binwalk decode_url2

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
15486         0x3C7E          PNG image, 1851 x 174, 8-bit/color RGB, non-interlaced
15527         0x3CA7          Zlib compressed data, default compression
```

There is another image hidden here, and potentially some compressed data?
Extracting with binwalk:

```bash
$ binwalk -D=".*" decode_url2

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
15486         0x3C7E          PNG image, 1851 x 174, 8-bit/color RGB, non-interlaced
15527         0x3CA7          Zlib compressed data, default compression
```

Viewing the hidden PNG image:

![Pasted image 20230617121633.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230617121633.png)

`flag{e62630124508ddb3952843f183843343}`

- Author : SpacerJa
