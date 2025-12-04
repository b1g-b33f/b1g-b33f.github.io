---
title: "Shady Oaks Financial Forceful Browsing"
date: 2025-12-3
author: Shawn Szczepkowski
---

Today we will be covering Shady Oaks Financial Forceful Browsing lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

The lab has a forceful browsing vulnerability. With this we will look for any high value endpoints that we probably aren't supposed to see, but the app just doesn't have the proper access controls in place.

For something like this we could generally use directory busting or crawling to discover hidden content. Robots.txt is also a great place to look, or maybe we would find a hidden endpoint in a JavaScript file, backup file, a PDF file etc. and during a live engagement we may also look at things like DNS records, shodan, or The Wayback Machine to help us discover content. In this case let's try a few common sense things like `/admin` and `/administrator` first before we work too hard.

Visiting `https://03d42aaeda7a.labs.bugforge.io/admin` immediately we see our flag in the admin panel, delivered by an API request to `/api/admin/flag`:
```http
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Thu, 04 Dec 2025 00:15:48 GMT
Etag: W/"30-TWDdmhW3BWrXk3a88xLdfIkTtBg"
X-Powered-By: Express
Content-Length: 48

{"flag":"bug{12e3d83dda00c1addeb288db2ba7cb5f}"}
```
