---
title: "Shady Oaks Financial Race Condition"
date: 2025-12-13
author: Shawn Szczepkowski
---

Todays lab is the Shady Oaks Financial Race Condition lab from [bugforge.io](https://bugforge.io). This is an easy rated lab.

While exploring the functionality of todays lab we notice that when converting currency there is a significant delay in the response indicating some back end logic is hard at work. 
This endpoint would be prime testing for a race condition.

Let's use one of the built-in custom actions with Burp Repeater.
![Setting up Custom Action](/assets/images/Pasted%20image%2020251209124838.png)
- Send to repeater
- 1 - open custom actions
- 2 - Use `Trigger race conditions` which send a basic 20 request single packet attack
- Now we have 2 options. We can click the `A` in figure 2 which would be `Attack on send` which will trigger when we send the repeater request or we can click the play icon to just start the attack. If we click play, which I did, you will only see the response codes and will need to click the edit icon to see the actual response.
- 3 - Observe response codes

Back in the application Observe that we have definitely converted more currency than we had. In our `/api/convert-currency` attack response we see our flag.
![Attack Results](/assets/images/Pasted%20image%2020251209130244.png)

![Manipulated Balance](/assets/images/Pasted%20image%2020251209125057.png)
