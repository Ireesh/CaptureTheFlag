\#Hacker101 CTF: A little something to get you started



We open the challenge page and see nothing except a welcome message and a white background.

The task should be easy, so we start with something simple - first, we open the page source code using CTRL+U.



<style>

&nbsp;   body {

&nbsp;       background-image: url("background.png");

&nbsp;   }

</style>



This seems excessive for a page with a white background. The image is located in the root directory, so we append /background.png to the original URL and get our flag.

