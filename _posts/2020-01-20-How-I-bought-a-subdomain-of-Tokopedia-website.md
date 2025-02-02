---
title: How i bought a subdomain of Tokopedia’s website (yeah you read it right)
tags: [Writeup, Bug Bounty, Hacking]
style: border
color: primary
description: A subdomain of tokopedia’s website is pointed to an expired Top-Level-Domain available to buy, so obviously I go ahead and buy it.
---

_Tl;dr :_ _a subdomain of tokopedia’s website is pointed to an expired Top-Level-Domain available to buy, so obviously I go ahead and buy it._

...

Greetings.

When digging some old emails, i found one of my findings on tokopedia.com, and I decided to make a write-up about it. Tokopedia is one of the few company from Indonesia, that host their own public Bug Bounty Program. You can read the rules and details here: [https://github.com/tokopedia/Bug-Bounty](https://github.com/tokopedia/Bug-Bounty).

After reading their rules, I noticed that the targets is mainly set to wildcard domains (*.tokopedia.com and *.tokopedia.net). So i began to enumerate the subdomain using a couple of open source tools like sublist3r, knockpy, massdns, etc.

I found some interesting subdomain where i found some interesting findings too ([The forgotten content : information disclosure and reflected XSS on Tokopedia](https://infosecwriteups.com/information-disclosure-and-reflected-xss-on-tokopedia-1b3a00ec64c6)). But the one that catches my eyes is REDACTED.tokopedia.com, because when i access the subdomain, i was getting an _ERR_NAME_NOT_RESOLVED_ error page from my browser.

Using **_dig_** command in my terminal, turns out that the CNAME configuration is pointing to another Top Level Domain (REDACTED.com).  So REDACTED.tokopedia.com is actually an alias for another different domain which is REDACTED.com.

Surprisingly, when i checked the whois record, the domain is actually expired. So i head straight up to namecheap.com to check if it’s available to buy, and well, it is.

![](https://miro.medium.com/v2/resize:fit:875/1*QKs2VfnGBIoppkEMXCs8oQ.png)

Here comes the dilemma, I was really broke at that time (seriously), i can’t even afford a domain for **_$8_**, **_PLUS_**, I’m still not sure whether this subdomain is actually takeover-able or not. I even consider to report this vulnerability without buying the domain, but i don’t think that’s a good proof of concept.

So, decided to take the risk, borrowing $8 from my friend (kudos to Mr. N), then bought the domain.

After pointing it to a free hosting service, i try to open the subdomain again, and well, i successfully taken over the subdomain because it’s now pointing to my own server.

![](https://miro.medium.com/v2/resize:fit:659/1*Ne3MNQWFseXwUTIRoDpBug.png)

I decided to make some PoC for stored XSS, but stumble upon a problem because the cookies is set with _secure_ flag, so the site needs to be hosted in https. After i set a free SSL certificate for REDACTED.com, another problem arise, the browser throws a privacy warning, because the site is accessed from REDACTED.tokopedia.com, while the certificate is signed for REDACTED.com.

![](https://miro.medium.com/v2/resize:fit:500/1*yj8Ogwe9UuaB5_yK20FiSg.png)

I got this from stackoverflow, the warning is different but you got the idea.

They won’t allow me to make a *.tokopedia.com SSL certificate since it’s reserved by the company, so i decided to report it right away. We could still steal cookies if the user decided to click ‘_proceed to…’_ button anyway, so yeah.

![](https://miro.medium.com/v2/resize:fit:875/1*43kD9p_iPgbVeGx4Z46kiA.png)

**_NOTE:_** _If you found a subdomain takeover, i think it’s not wise to show something unnecessary on the front page, the company won’t like it because users could saw it, and it’s bad for their reputations. This report is from my earlier days as a Bug Bounty Hunter, so showing “Subdomain taken over” on front page might not be the best idea. Instead, we could write our PoC on hidden html tag, and maybe showing “Under Maintenance” on the front page. And don’t forget to randomize the file name we hosted on that subdomain so it won’t be accidentally found by users._

Well, the Security Team verified my report, and it’s a valid security bug with High Severity, and they decided to reward me.

Spending 8 bucks for three digits bounty rewards? I see this as an absolute win.

![](https://miro.medium.com/v2/resize:fit:875/1*Y6FvvxnlVerizmgYJhn9Vw.jpeg)

Thanks.

**Timeline:**

-   **July 27 2019:** Report sent.
-   **July 29 2019:** Security team verified the report, valid with High Severity.
-   **July 31 2019:** Bug fixed, they asked me to re-test the bug.
-   **November 20 2019:** $$$ awarded.
