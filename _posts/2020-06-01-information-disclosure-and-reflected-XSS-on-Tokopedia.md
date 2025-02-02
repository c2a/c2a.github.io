---
title: "The forgotten content : information disclosure and reflected XSS on Tokopedia"
tags: [Writeup, Bug Bounty, Hacking]
style: border
color: primary
description: How I found a ‘should be deleted’ content that disclose some sensitive information and vulnerable to reflected XSS on Tokopedia’s website.
---

_Tl;dr : how i found a ‘should be deleted’ content that disclose some sensitive information and vulnerable to reflected XSS on Tokopedia’s website._

...

Greetings.

On my previous write-up about subdomain takeover on Tokopedia, I mentioned that i also found another interesting subdomain, one of them is _accounts-REDACTED.tokopedia.com_.

Because of the interesting subdomain name, i decided to dig some more in this particular subdomain. Well, the front page only shows **410 Gone** http code and nothing else, so it means that the resource on this subdomain has been intentionally removed and it should be purged.

I didn’t stop there, i used some tools for directory brute-force, dirbuster, dirsearch, etc. and all the brute-forced path returns **410 Gone**, except this one  path, _accounts-REDACTED.tokopedia.com/docs/_, which returns different response, a **200 OK** http code.

I opened the path on my browser, but it returns **410 Gone**.

![](https://miro.medium.com/v2/resize:fit:875/1*3ck1KSO_my7JzsPbpkiaIw.jpeg)

I’m pretty sure it’s not a false-positive. After double checking, turns out that it’s served on different port. Port **80 (http)** returns **200 OK**, while port **443 (https)**returns **410 Gone**, and my browser automatically redirect to https if i don’t specify it directly.

So i open the path using **http** protocol and i noticed that the _/docs/_ path on port **80** is an API documentation using Swagger UI, an open source project to visually render documentation for an API defined with the OpenAPI (Swagger) Specification.

![](https://miro.medium.com/v2/resize:fit:875/1*l6v0n7yHBllc3jRb3Z4GvQ.png)

Surprisingly, it contains some sensitive API like retrieving user data, and other sensitive PII like National Identity Number, etc. and it was meant for third party partner of Tokopedia. It even contains the client ID and Client secrets for Authorization.

![](https://miro.medium.com/v2/resize:fit:623/1*feTyBpnCJReRvq5xAUjyDw.gif)

BUT, after i check it out, the API isn’t working anymore. Actually, on my previous experience hunting on Tokopedia, they‘re using GraphQL now, different from this one i found. I checked the changelog, and last update from their staffs are from around Juny 2019. I was late, 2 months late and they’ve changed their web architecture. The **410 Gone** status code makes sense now, this resource was supposed to be deleted, but still there somehow.

![](https://miro.medium.com/v2/resize:fit:875/1*3Tx7a91Flz6iFokj7Gi3vg.jpeg)

So i started hunting for CVE in this particular version of Swagger UI. I actually found one XSS CVE, but after i try it, yet another disappointment, it’s not working.

![](https://miro.medium.com/v2/resize:fit:850/1*H2NsulI7l1Nu0PQAoYtIng.jpeg)

It got me really curious, _why is it not working?_ because this version supposed to be vulnerable.  After some more digging, i think they’ve customized this swagger UI, they even put some credit of their staff’s name on the partially exposed source code. So _maybe_, this swagger UI is different from the official version.

**BUT I‘M NOT GIVING UP —** _Well, actually i did take a break from this case for about 2 weeks._

After that I started to dug more. I noticed that this documentation is open, means anyone can modify or update the documentation without having to login or register. I actually accidentally deleted the whole documentation once, but fortunately i can download the past data from the changelog and import it again.

![](https://miro.medium.com/v2/resize:fit:281/1*mCX65MYG1UAq-Yd9cPjTlg.jpeg)

I found this interesting URL from import functionality :

http://accounts-REDACTED.tokopedia.com/docs/index.php?url=_file.yaml_

I tried to look for LFI and SSRF on the url parameter, nothing works. But i noticed that the file name is reflected on the html page, so i tried to insert XSS payload, yet another disappointment, **no** pop up appear.

But when I did inspect element to check the reflected result, turns out that the XSS is actually fired, but blocked by Chrome’s XSS auditor. It works perfectly on another browser like Firefox etc.

![](https://miro.medium.com/v2/resize:fit:875/1*F9Isu7SR7tz2o5PapphE4g.png)

The final payload looks like this:

`http://accounts-REDACTED.tokopedia.com/docs/index.php?url=['<script>alert(document.cookie)</script>']`

I tried this on the latest version of Swagger UI, but it’s not working, and I can’t find any CVE documenting this vulnerability on older version. So _maybe_ I’m right, their version is already customized (or maybe nobody assign a CVE yet? or I missing something here ¯\_(ツ)_/¯ ).

Anyway I reported this to their security team and got response **_20 minutes later_** told me that this report is verified with medium severity. Pretty good response time.

Thanks.

**Timeline:**

-   **August 29 2019:** Report sent.
-   **August 29 2019:** Security team verified the report, valid with Medium Severity.
-   **September 17 2019:** Bug fixed, they asked me to re-test the bug.
-   **November 20 2019:** $$$ awarded.

_NB: Pardon me for the excessive use of Memes, we need more smile in this hard situation. Stay safe and wash your hands! :D_
