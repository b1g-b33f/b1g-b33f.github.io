---
title: "Sokudo Broken Access Control Weak Token"
date: 2025-12-4
author: Shawn Szczepkowski
---

Today we will be covering the SoKudo Broken Access Control Weak Token lab from [bugforge.io](https://bugforge.io). This is a Medium rated lab.

While exploring the lab we notice immediately that the Token appears to be only a time stamp.
```txt
Authorization: Bearer 20251205000042
```

A token like this  has very weak entropy and could easily be bruteforced, but being a time stamp let's also draw some attention to another interesting endpoint `/api/stats/leaderboard `. In the response we can see the last login information of each user. What if this time stamp token is based off that? We could test our users token to see if it matches our last login, but since this is so easy to try let's create one based off the admin users last login.
```sh
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Fri, 05 Dec 2025 00:03:26 GMT
Etag: W/"188-AOIQmoj8Rmqr5kIbY5KIevCN1a4"
X-Powered-By: Express
Content-Length: 392

[{"username":"admin","best_wpm":null,"total_sessions":null,"avg_wpm":null,"last_login":"2025-12-05T00:00:42.494Z"},

...snip...
```

The token would look like this:
```txt
Authorization: Bearer 20251205000042
```

Let’s convert step-by-step:

- Year: **2025**
    
- Month: **12**
    
- Day: **05**
    
- Hour: **00**
    
- Minute: **00**
    
- Second: **42**

Now, we could go about using this token many ways. But let's just replace our token in local storage with our made up token. Refresh the page and we have our Admin panel/flag.
![Admin Panel](/assets/images/Pasted image 20251204191739.png)
