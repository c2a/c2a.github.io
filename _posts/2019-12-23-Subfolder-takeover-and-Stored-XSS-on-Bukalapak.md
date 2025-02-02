---
title: How Inspect Element lead to Subfolder takeover and Stored XSS on Bukalapak’s website
tags: [ Writeup, Bug Bounty, Hacking ]
style: border
color: primary
description: A unique high severity misconfiguration I found on Bukalapak website that lead to Subfolder takeover and stored XSS, by only inspecting an HTML element on their webpage.
---

_Tl;dr : a unique high severity misconfiguration I found on Bukalapak website that lead to Subfolder takeover and stored XSS, by only inspecting an HTML element on their webpage._

...

Greetings.

Bukalapak is one of the biggest online marketplace and “unicorn” startup located in Indonesia. One day when I was taking a break and checking their website to buy something, I noticed that they held a Bug Bounty Program and I think it would be cool if I could carve my name on their lovely “[**Wall of fame**](https://bukalapak.github.io/bukabounty/)”.

I’m especially interested to look for vulnerability on one of their new feature that is hosted on a specific subdomain, _REDACTED_.bukalapak.com. Simply because it’s a new feature, so I think it’s more likely that they missed something which could lead to a vulnerability.

Long story short, after some time, I couldn’t find anything interesting beside some minor or very low severity bug like clickjacking with no sensitive action, rate-limiting issue, etc.

But, when I did inspect element on one of the pages to check if my XSS payload fired or not (**_it’s not, sadly_**), I found something that catches my eye on the browser console. The page is trying to fetch an image on another subdomain, but failed and return a 404 response printed on the browser console. The URL looks like this:

https://REDACTED.bukalapak.com/img/some-random-text.jpg

![](https://miro.medium.com/v2/resize:fit:604/1*tyvx74849u-AcWaDoBczBA.png)

Broken link

I got curious and opened the link on a new tab, and surprisingly, i got that beautiful “NoSuchBucket” error page from Amazon S3 along with the bucket name.

![](https://miro.medium.com/v2/resize:fit:875/1*he_8jAG8dZzwcIBjJbWAjQ.jpeg)

Beautiful…

At this point, I know that takeover is mostly possible, but I’m curious because previously, all the tools related to subdomain takeover scanner that I used can’t detect this. So I strip the URL to find the main address that pointed to the unclaimed Amazon S3 Bucket. I found out that the URL is something like this :

https://REDACTED.bukalapak.com/img/

Turns out that the _REDACTED_.bukalapak.com is up and well, it host another feature from bukalapak website and work beautifully. That’s why I decided to write this as “_subfolder takeover_” and not “_subdomain takeover_” because I took over a _subfolder_ and not a _subdomain_, although it has the same methodology.

After taking a sip of my coffee, I started the takeover process, i made an Amazon S3 Bucket with the name printed on the error page. When choosing a region, [**Patrik Hudak**](https://0xpatrik.com/) on his blog actually have wrote about how to guess the region (_he wrote a lot of_ [_amazing articles_](https://0xpatrik.com/) _about subdomain takeover, you should read them if you have the time_), but considering bukalapak is a product from Indonesia, I decided to just choose the nearest possible region (Asia Pacific), and turns out I was right. Take over is complete, the subfolder is now pointed to my controlled Amazon S3 bucket.

_So, what could I achieve by taking over this subfolder?_

The first one comes to my mind is stored XSS, i found out that their cookies is set to a wildcard subdomain, so basically they used the same cookies everywhere, XSS to steal session cookies is possible. The second one, because this subfolder is hosted in one of their subdomain, clickjacking is possible on any page with X-Frame Options set to same origin subdomain, which most of the times contain very sensitive actions. I could also host a phishing content too, which if combined with the XSS and clickjacking, could be a very powerful attack vector.

![](https://miro.medium.com/v2/resize:fit:570/1*85hkA5PECb6eD3bgFfHAEA.png)

The Popup everyone loves.

But I didn’t exploit further because I’m afraid it’s against their rule, so I decided to report it right away and let them decide the severity.

Surprisingly, their Cyber Incident Responder reply my report within less than half an hour! very cool response time, kudos to their security team. They asked me to upload a specific file to confirm my findings so I do it right away.

Thanks.

**Timeline:**

-   **August 13 2019**: report sent.
-   **August 13 2019 (Less than half an hour later)**: Cyber incident responder reply my email, asking me to upload a specific file to confirm my findings.
-   **August 14 2019**: report validated, categorized as misconfiguration with High severity level.
-   **August 26 2019**: they carved my name on their Wall of Fame.
-   **September 24 2019**: $$$ paid with a thank you note.
-   **December 23 2019**: disclose request approved by security team, write-up published.
