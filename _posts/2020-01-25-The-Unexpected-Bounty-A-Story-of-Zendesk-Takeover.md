---
title: "The Unexpected Bounty: A Story of Zendesk Takeover on REDACTED.com"
tags: [Writeup, Bug Bounty, Hacking]
style: border
color: primary
description: "A good faith powered report of subdomain takeover that ends up with a bounty, even though the company itself doesn’t have a Bug Bounty Program."
---

_Tl;dr : a good faith powered report of subdomain takeover that ends up with a bounty, even though the company itself doesn’t have a Bug Bounty Program._

...

Greetings.

It all started with a Linkedin Connection Request.

The Person‘s bio says that she work as a Talent Acquisition Specialist on a company, _REDACTED.com_. Me, after got a duplicate on my previous report about XSS on Google, decided to take a break from there and check this _REDACTED.com_ instead.

After some subdomain enumerations, i found something interesting on _support.REDACTED.REDACTED.com_. When i tried to access the mentioned subdomain, i got the following page:

![](https://miro.medium.com/v2/resize:fit:875/1*CtjJmOVl_iYsBjI9vESx2w.png)

It looks like the subdomain is pointing to a zendesk help center page which is not claimed or no longer exists. Using **_dig_** command, I got the CNAME record.

![](https://miro.medium.com/v2/resize:fit:875/1*NpUbve7yYz0I22-yB0qiBg.png)

After reading Zendesk documentation, i successfully register a new account and taken over the subdomain. I was also able to get stored XSS by enabling the SSL to stop the redirect, then make a guide html page with an xss payload.

![](https://miro.medium.com/v2/resize:fit:703/1*Q3QlnQau7aLpdOO0LkP9Lw.png)

I didn’t report it immediately, because they don’t have Bug Bounty Program and i can’t find any contact related to security on their website. Days later, i received a couple of tickets (around 10) from their customers, turns out this zendesk portal is still being used, and tickets from their customers is being forwarded to this portal from the main(another) website.

![](https://miro.medium.com/v2/resize:fit:849/1*ls-n1R3acjTgLYNj1iaT7A.png)

![](https://miro.medium.com/v2/resize:fit:638/1*KbHDOGIDya-lUxOgQ5GiAg.png)

hmm…

I decided to ask the Talent Acquisition Specialist from the Linkedin, where Could I report this vulnerability? She gave me an Email of their Security Team and I immediately report this vulnerability because my email flooded with tickets from their customers.

I was just being a good guy and sent this report without expecting anything in return because I know they didn’t have a Bug Bounty Program, and a simple “Thank You.” would be very enough.

But to my surprise, they decided to rewards me with bounty. Well, the unexpected money is the best money.

![](https://miro.medium.com/v2/resize:fit:875/1*gL2I3xPVzNwxNGspXSwKyQ.png)

This is the fastest bounty I’ve ever received so far (more or less a week after report sent), and also mark my first bounty in 2020.

Thanks.

**Timeline:**

-   **January 17 2020:** Report sent.
-   **January 20 2020:** Report validated with High severity.
-   **January 23 2020:** $$$ paid, limited disclosure request approved.
