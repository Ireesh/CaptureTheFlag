# Hacker101 CTF: Micro-CMS-v2

### Flag №1:

Begin with the simplest approach. We discovered the site and found that common links have edit capability. However, for this site we need to log in. Let's examine the request. There we can see the `GET` method and the `302` status code, which suggests trying other request methods. Click `Copy as fetch` on the request in the `Network` tab of `DevTools`, paste it into the `Console`, change the method to `POST`, and send it. We get status code `200`, go to the `Response` tab to obtain the **FIRST FLAG**.

### Flag № 2:
In the first version of the task, there was an SQL injection vulnerability, so let's try to find it here. After several attempts to insert different injection payloads into the URL field used for page navigation, no results were obtained. However, there is another input field - the login page. Typing ` ' ` returns a `500 Server Error`, indicating that the server processed the request but we didn't provide valid parameters. This marks the beginning of the brute-force injection process. We determined we were dealing with a blind SQL injection.

```mysql
‘ OR 1=1 — ‘
```
We received an `Invalid password` message, meaning we passed the username check. Attempting the same with the password was unsuccessful. Let's use the hint provided. One hint suggests using `UNION SELECT`. Let's try that:

``mysql
username:
‘ union select ‘admin’ – ‘

password:
admin
```

Gotcha! We are logged in. The first thing we notice is the `Private page` link. Going there reveals the **SECOND FLAG**.

### Flag №3

For the third task, I attempted XSS, which was still functional but didn't yield a flag, so I decided to investigate the database since we have access through SQL injection. This time, we'll use an automated tool called SQLmap.

```bash
sqlmap -u "https://09fa58cae31ac771fad86193ee44ee58.ctf.hacker101.com/login" \
  --data="username=admin&password=admin" \
  --batch \
  --dump          
```

This provided us with the database structure. The database name is level2, containing two tables: admins and pages. The first table, admins, is of interest and contains the following columns: id, username, and password.

```bash 
sqlmap -u "https://09fa58cae31ac771fad86193ee44ee58.ctf.hacker101.com/login" \
  --data="username=admin&password=admin" \
  --batch \
  --sql-query="SELECT * FROM level2.admins"
```

<img width="754" height="361" alt="Снимок экрана 2026-01-13 190700" src="https://github.com/user-attachments/assets/441697b1-1fb2-4cc8-b7fe-905f740d794c" />


The table contains only one user: 1, marnie, laree. Attempting to log in with these credentials fails. Referring back to the previous solution, we try using the username as the password:

```html
username: marnie
password: marnie
```
This gives us the **THIRD FLAG**! This indicates that the authorization logic also had a flaw: it compared the username with the user-typed password.



