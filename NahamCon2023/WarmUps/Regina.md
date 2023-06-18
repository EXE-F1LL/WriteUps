>Author: @JohnHammond#6971  
>  
>I have a tyrannosaurus rex plushie and I named it Regina! Here, you can talk to it :)
---------------------------------
Entering the challenge, we are greeted with a banner:

```bash
/usr/local/bin/regina: REXX-Regina_3.9.4(MT) 5.00 25 Oct 2021 (64 bit)
```

A quick google search shows this is a rather popular interpreter that I have never heard of. It has a very thorough man page located [here](https://master.dl.sourceforge.net/project/regina-rexx/regina-documentation/3.9.4/regina.pdf?viasf=1). However, no matter what my input is, I cannot seem to get a response. I learned the proper syntax that should work for testing purposes, but nothing connects back:

```bash
/usr/local/bin/regina: REXX-Regina_3.9.4(MT) 5.00 25 Oct 2021 (64 bit)
Say "Test"

client_loop: send disconnect: Broken pipe
```

After trying things for quite a while, right before giving up, I miraculously stumbled upon a response somehow:

```
/usr/local/bin/regina: REXX-Regina_3.9.4(MT) 5.00 25 Oct 2021 (64 bit)
'say test'
;
run
'say test';
Say "Test";
exec
run
;
go
^F
mysh: say: not found
 
^Fmy
my
sh: RUN: not found

sh: say: not found
Test
 
sh: EXEC: not found

sh: RUN: not found
Ssh: GO: not found
Connection to challenge.nahamcon.com closed.
```

While just enough to keep me from giving up, I had a lot of trouble recreating this still. While I had pressed \^F in the cli, I was under the impression that I also had pressed \^D and this keystroke was important for the execution. However, simply pressing \^D on its own ended up terminating the `ssh` connection.

Eventually I landed on a goofy pattern to execute, where I write my commands, quickly press the `enter` key two times, immediately followed by \^D, then another `enter` stroke. With this, my commands would be executed and then the connection terminated.

```bash
/usr/local/bin/regina: REXX-Regina_3.9.4(MT) 5.00 25 Oct 2021 (64 bit)
ADDRESS SYSTEM "ls"
Say "Test"


  
flag.txt
Test
Connection to challenge.nahamcon.com closed.
```


```bash
/usr/local/bin/regina: REXX-Regina_3.9.4(MT) 5.00 25 Oct 2021 (64 bit)
line_str = linein("flag.txt") 
say line_str
  
flag{2459b9ae7c704979948318cd2f47dfd6}
Connection to challenge.nahamcon.com closed.
```

CTRL+D is commonly used for exiting programs, so it is somewhat sensible that it finalizes REXX-Regina for execution. Was this intended, though? It certainly felt like a strange way to interact.

- Author : SpacerJa
