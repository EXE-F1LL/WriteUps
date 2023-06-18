>Author: @JohnHammond#6971  
>  
>Oh, shoot, I could have sworn there was a flag here. Maybe it's still alive out there?
-------------------------------
Logging in, we see an interesting .user-entrypoint.sh file:
```bash
user@zombie:~$ ls -al
total 24
drwxr-sr-x    1 user     user          4096 Jun 15 22:23 .
drwxr-xr-x    1 root     root          4096 Jun 14 17:52 ..
-rwxr-xr-x    1 user     user          3846 Jun 14 17:52 .bashrc
-rw-r--r--    1 user     user            17 Jun 14 17:52 .profile
-rwxr-xr-x    1 root     root           131 Jun 14 17:52 .user-entrypoint.sh
```
the contents:
```bash
user@zombie:~$ cat ~/.user-entrypoint.sh 
#!/bin/bash

nohup tail -f /home/user/flag.txt >/dev/null 2>&1 & # 
disown

rm -f /home/user/flag.txt 2>&1 >/dev/null

bash -i
```
This looks like something that might be run upon entry. Just to check we can see our default terminal in `/etc/passwd`:
```bash
user@zombie:~$ cat /etc/passwd | grep user
user:x:1000:1000:Linux User,,,:/home/user:/home/user/.user-entrypoint.sh
```
Looks like this did indeed run and we can see very clearly the `rm` function getting rid of the flag. oops.
Also notable is the `tail -f` . The -f flag is a live following, or reading the final lines of the file.  We can also see a trailing &, meaning this has been set to background. As for `nohup`:
```bash
$ nohup
BusyBox v1.27.2 (2018-06-06 09:08:44 UTC) multi-call binary.

Usage: nohup PROG ARGS

Run PROG immune to hangups, with output to a non-tty
```
We can see this prevents accidental closings. This process is likely still running.
```bash
user@zombie:~$ ps aux
PID   USER     TIME   COMMAND
    1 root       0:00 /usr/sbin/sshd -D -e
    7 root       0:00 sshd: user [priv]
    9 user       0:00 sshd: user@pts/0
   10 user       0:00 {.user-entrypoin} /bin/bash /home/user/.user-entrypoint.sh -c bash
   11 user       0:00 tail -f /home/user/flag.txt
   13 user       0:00 bash -i
   18 user       0:00 bash
   25 user       0:00 ps
```
We do indeed see this process still running with a PID 11. My thinking is perhaps we can see the contents by exploring the PID in /proc/
```bash
user@zombie:~$ cd /proc/11/
user@zombie:/proc/11$ ls
arch_status      clear_refs       cpuset           fd               limits           mem              net              oom_score        projid_map       setgroups        stat             task             uid_map
attr             cmdline          cwd              fdinfo           loginuid         mountinfo        ns               oom_score_adj    root             smaps            statm            timens_offsets   wchan
auxv             comm             environ          gid_map          map_files        mounts           numa_maps        pagemap          schedstat        smaps_rollup     status           timers
cgroup           coredump_filter  exe              io               maps             mountstats       oom_adj          personality      sessionid        stack            syscall          timerslack_ns
```
`fd` refers to file descriptors, which might contain info from stdin, stdout, etc.
```bash
user@zombie:/proc/11$ cd fd
user@zombie:/proc/11/fd$ ls
0  1  2  3
user@zombie:/proc/11/fd$ ls -al
total 0
dr-x------    2 user     user             0 Jun 15 22:42 .
dr-xr-xr-x    9 user     user             0 Jun 15 22:41 ..
lr-x------    1 user     user            64 Jun 15 22:43 0 -> /dev/null
l-wx------    1 user     user            64 Jun 15 22:43 1 -> /dev/null
l-wx------    1 user     user            64 Jun 15 22:43 2 -> /dev/null
lr-x------    1 user     user            64 Jun 15 22:43 3 -> /home/user/flag.txt (deleted)
```
While we do see that it notes flag.txt is deleted. However, perhaps we can still see something, since the program is still in running memory?

```bash
user@zombie:/proc/11/fd$ cat 3
flag{6387e800943b0b468c2622ff858bf744}
```
And the flag is here. Clearly a simple `rm` might not be enough to exterminate the file out of existence. However, this situation is probably a little unusual since the `tail -f` was still running long after the rm command.

`flag{6387e800943b0b468c2622ff858bf744}`
