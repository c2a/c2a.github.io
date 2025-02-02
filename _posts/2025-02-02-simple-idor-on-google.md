---
title: "Google did an Oopsie: a simple IDOR worth $3,133.7"
tags: [Writeup, Bug Bounty, Hacking, Google]
style: border
color: primary
description: Sometimes the bounty is hidden in plain sight — a simple IDOR by changing the Google Drive file ID.
---

**_Tl;dr_**_: Sometimes the bounty is hidden in plain sight — a simple IDOR by changing the Google Drive file ID.

…

**_Pssst_**_… I am now part of_ [**_SysBraykr_**](https://sysbraykr.com)_, an offensive security company from Asia. Go check out our website if you want to meet a bunch of interesting people in the cybersecurity field (like me! :))._

…

Back in 2019, when I had just started learning about hacking and bug bounty hunting, I had this dream of being in Google’s Hall of Fame.

I mean, their products are everywhere — billions of people use them every day — so I thought it would be awesome to be one of the recognized people who helped make it safer for everyone. Back then, there weren’t that many people doing it.

So, I randomly chose one of their products and started hunting. As expected, it wasn’t that easy. Days went by, and nothing came up. I started to think maybe I wasn’t ready for this and should go for an easier target. But then, I decided to switch to a considerably less famous product of Google : **_REDACTED.google.com_**.

After reading their documentation on _developer.google.com_ to understand the functionality, I began my hunt. On one occasion, I actually found an XSS vulnerability, but it triggered on a properly sandboxed domain (_*.googleusercontent.com_), which is intended behavior and not a valid bug according to [their rules](https://bughunters.google.com/learn/invalid-reports/web-platform/xss/6619189462433792/xss-in-sandbox-domains).

That was until I stumbled upon a feature where we could import a file from Google Drive (and it worked perfectly). When I intercepted the request using my lovely Community Edition Burp Suite, I noticed that it was using the drive file ID in the post parameter **_docId_** to identify which drive file we chose.

![](https://cdn-images-1.medium.com/max/1000/1*hH6nt4vTZBPvqDU6Ad0pDQ.png)

I noticed that the **_docId_** is present in the Google Drive file URL, something like this:

_https://drive.google.com/open?id=_**_18TrUTt3SI3fmKNut8SREDACTED_**

It returns a JSON response with a token and the title of the file we picked. I sent it to the repeater and started to play with the request. A crazy thought suddenly crossed my mind — _what if I used someone else’s file? A private one?_

![](https://cdn-images-1.medium.com/max/1000/1*jJ6fiJMex_kzfPTPlc7JtA.jpeg)

hmm…

For those who don’t know, we can make a file private on Google Drive by changing the permission settings. If other people want to access it, they have to request access from the owner.

![](https://cdn-images-1.medium.com/max/1000/1*Hyh68Vmy8ZIwkWqfxlCliA.png)

So, I created a private file on my other account and put the ID in the **_docId_** parameter. And surprisingly, it worked! The server returned a 200 response along with the token and file name, even though the requester shouldn’t have had access to the private file.

![](https://cdn-images-1.medium.com/max/1250/1*mb4D5CaGntQSFCb3DZOALQ.png)

![](https://cdn-images-1.medium.com/max/500/1*fCP0TE1NwyypBSmDogc16w.jpeg)

Did i just find an IDOR?

I reported this bug straight away. The next day, they triaged the report and escalated it from P4 all the way to P2. Two days later, I got the beloved “Nice Catch!” catchphrase.

![](https://cdn-images-1.medium.com/max/1000/1*jqd0lOcma76BRUdn3IqiYA.png)

Nice Catch!

At that time, although the bounty decision was still in discussion, my name had already been carved into their “**_Honorable Mention_**” page (not **_Hall of Fame_**, not yet). I honestly didn’t expect to get a bounty reward, considering this was a fairly simple, low-hanging fruit bug. I needed to know the file ID to exploit it, which is a fairly unique, long, random token, **_AND_** I could only go as far as knowing the file name and file type.

But I think they take security very seriously (_or maybe they found something worse internally — who knows ¯_(ツ)_/¯_). Because, surprisingly, three weeks later, I got an update saying my report was worth $3,133.70. Damn, they’re crazy (in a good way, of course ❤).

![](https://cdn-images-1.medium.com/max/1250/1*E_y3JbFtWqxCEi0ChUV_kQ.png)

![](https://cdn-images-1.medium.com/max/500/1*la8BcATdxvmC1dmn6ds_-w.gif)

Timeline:

-   **_September 8, 2019_**: Vulnerability reported.
-   **_September 9, 2019_**: Report triaged, escalated from P4 to P2.
-   **_September 12, 2019_**: Nice Catch!
-   **_October 1, 2019_**: Update regarding bounty, and they put my name on the Hall of Fame. Nice!
-   **_November 14, 2019_**: Bounty paid.
