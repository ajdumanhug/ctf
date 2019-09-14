# Challenge Name: Unagi
Score: 200 <br>
Vulnerability: External XML Entity (XXE) <br>
Solution: Bypass character encoding <br>
Link: http://web.chal.csaw.io:1003

**Part 0x01 - Information Gathering** <br>
There are three interesting pages on the website: [user](http://web.chal.csaw.io:1003/user.php), [upload](http://web.chal.csaw.io:1003/upload.php), and [about](http://web.chal.csaw.io:1003/about.php). This site is written using PHP based on the file extensions of the pages.

Displayed information of users on `user.php` page are `name`, `email`, `group`, and `intro`. In `upload.php` page, there's a format sample in XML. (http://web.chal.csaw.io:1003/sample.xml)
```
<users>
  <user>
    <username>alice</username>
    <password>passwd1</password>
    <name>Alice</name>
    <email>alice@fakesite.com</email>
    <group>CSAW2019</group>
  </user>
  <user>
    <username>bob</username>
    <password>passwd2</password>
    <name> Bob</name>
    <email>bob@fakesite.com</email>
    <group>CSAW2019</group>
  </user>
</users>
```
About.php page gives us a hint that the flag is located at `/flag.txt`.

**Part 0x02 - Vulnerability Exploitation** <br>
We can perform an External XML Entity (XXE) attack on this challenge.
![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-14%20at%2011.51.22%20PM.png)
Adding the following payload `<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>` throws a WAF error message.
![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-14%20at%2011.56.02%20PM.png)
So to solve this, we have to look for WAF bypass to exploit the vulnerability.

Searching on Google gives us this article: https://www.phdays.com/en/press/news/phdays-vi-waf-bypass-contest/. Check out #3 for XXE WAF Bypass. 
<br>

The author found out that if we converted the character encoding of UTF-8 to UTF-16 Big Endian, we could easily bypass the WAF.
> encoded the body in UTF-16 Big Endian via the command cat x.xml | iconv -f UTF-8 -t UTF-16BE > x16.xml
<br>
Using the sample.xml given by the challenge author, we will convert the character encoding of the file to UTF-16BE.

![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-15%20at%2012.04.27%20AM.png)

After uploading the XML file, we managed to retrieve the `passwd` file from the internal server. However, it only prints 20 characters.
![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-15%20at%2012.19.54%20AM.png)

I tried adding the variable to Name and Email but the same result. Fortunately, I remembered that there's another field named `intro` in the user page. I then added it to the XML file and converted the character encoding again.

![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-15%20at%2012.18.19%20AM.png)

Uploading it to the challenge website will give us the content of the `passwd` file.

![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-15%20at%2012.24.21%20AM.png)

But that doesn't mean we can quickly get the `flag` located at `/flag.txt`. To get the flag, I have to use the `php://filter` wrapper.

Final Payload would be 
```
<!DOCTYPE root [<!ENTITY test SYSTEM 'php://filter/convert.base64-encode/resource=/../flag.txt'>]>
```

Uploading it will print a BASE64 cipher.

![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-15%20at%2012.33.08%20AM.png)

Decoding it to text will give us the flag.

![](https://github.com/ajdumanhug/ctf/blob/master/web/xxe/files/Screen%20Shot%202019-09-15%20at%2012.35.00%20AM.png)

Thanks for reading.
