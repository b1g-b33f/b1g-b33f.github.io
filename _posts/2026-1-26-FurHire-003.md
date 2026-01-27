---
title: "FurHire-003 Second Order SQLi"
date: 2026-1-26
author: Shawn Szczepkowski
---

In todays lab we have FurHire and app that let's you sign up as a pet looking to provide services, or a recruiter looking to hire a new pet. As any good tester would do, make an account for each and explore both roles, and also how they interact with each other.


There is quite a bit to explore in this app, but I'll cut to the chase. During input testing, I noticed I could post a job as a recruiter with just a single quote in the Job Title:
![FurHire1](/assets/images/Pasted%20image%2020260126181108.png)

The job posts successfully and nothing looks off when we view the job or applicants to the job:
![FurHire2](assets/images/Pasted%20image%2020260126181340.png)

But if we are paying close attention, when we do actually get an applicant, and we `View Applicants` we will see there are no applicants. What gives?
![FurHire3](assets/images/Pasted%20image%2020260126181553.png)

Let's try something else. Let's post a job called `' or 1=1` and see what happens:
![FurHire4](assets/images/Pasted%20image%2020260126182002.png)

No error so everything must be ok and the app is just being weird? If we are testing inputs, it's important to see what is happening to those inputs elsewhere like we did earlier. Let's view applicants for that job and see what happens:
![FurHire5](assets/images/Pasted%20image%2020260126182147.png)

We have a database error! Let's try and comment out our `' or 1=1` like this `' or 1=1--` and see what happens:
![FurHire6](assets/images/Pasted%20image%2020260126182308.png)
- We now see we have applicants for this position and we really don't, plus our database error is gone.

I'll save you some of the pain I went trough here, but order by doesn't play well, and the back end strips commas.

Let's see if we can determine the database we are up against using `'or(SELECT 1 FROM sqlite_master)--`:
![FurHire7](assets/images/Pasted%20image%2020260126182616.png)
- `sqlite` returns true

Now I was able to use some payloads to help determine tables also:
```sql
'or(SELECT 1 FROM users)--

'or(SELECT 1 FROM config)--
```

Users did not prove to have anything of interest but if we explore config a bit we can see there is a key called `flag`:
```sql
' OR EXISTS(SELECT 1 FROM config WHERE key='flag')--
```

I'll save you some pain here also. I pulled out the flag char by char and it took FOREVER. If it wasn't simple but painful I would have attempted to script it.
```sql
'or(SELECT 1 FROM config WHERE key LIKE '%flag%' AND value GLOB 'bug{*')--
```
![FurHire8](assets/images/Pasted%20image%2020260126183206.png)

I learned after the fact that you can get a `UNION` operator to do some work here even with the commas being stripped it would just take some trial and error on your part since `orderby` is no help:
```sql
' UNION SELECT * FROM (SELECT value FROM config WHERE key='flag') JOIN (SELECT 2) JOIN (SELECT 3) JOIN (SELECT 4) JOIN (SELECT 5) JOIN (SELECT 6) JOIN (SELECT 7) JOIN (SELECT 8) JOIN (SELECT 9) JOIN (SELECT 10) JOIN (SELECT 11) JOIN (SELECT 12) JOIN (SELECT 13) JOIN (SELECT 14) JOIN (SELECT 15)--
```
![FurHire8](assets/images/Pasted%20image%2020260126183653.png)
