---
title: "G Suite and Cloudflare"
datePublished: Mon Oct 16 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0itupq00ygzfs17pyd1qz3
slug: g-suite-and-cloudflare
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380899763/r4GnKMSFp.jpeg
tags: security

---


Recently, I founded my own company and to send email from my own domain, I use [G Suite from Google](https://gsuite.google.com/features/). Here is how I set up my company email a few weeks ago and a few important points to take into account.

After I click on the big _Get Started_ button on the G Suite home page, I'm welcomed, told I'm getting a 2 week free trial and then I'm off to set up my G Suite. Most of it is just choosing options, so I will only add comments for the "hardest" parts.

![1-lets-get-started.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896065451/zF82pPLy4.jpeg)

![2-business.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896212224/V7Rqc2Vb0.jpeg)

![2-business1.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896222145/MnZrC2obo.jpeg)

![3-location.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896126239/4Ski6Mgy-.jpeg)

![4-contact.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896143337/COdfS-5MV.jpeg)

![6-domain-name.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896198602/9Y54xnwAu.jpeg)

![7-admin.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896241906/xGNJUyjI0.jpeg)

Once I filled out the basic information, the first important point arose:

![8-sign-in](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380895585/brAVTAhV6.jpeg)

This is the account I will use for my company, this is directly relevant to how much I earn, how I present myself and will grant access to a lot of private and important information. I want to have a very secure password. That is why I use [1Password](https://1password.com/) to generate a very long, very random password. Later, when I have access to my account, I will also set up [2 factor authentication](https://www.google.com/landing/2step/).

My email, personal or professional, is the gateway to all my other accounts. I want to protect this as best as I can. That is why I take these security measures and I strongly suggest you do to. I cannot stress enough how important it is to keep email as secure as you can. Although I'm a huge fan of how [1Password](https://1password.com/) handles passwords, logging in and locking the vault at regular intervals, I also understand it's not the cheapest option out there. If this is a concern for you, check out [LastPass](https://www.lastpass.com/), [KeePass](https://keepass.info/) or [DashLane](https://www.dashlane.com/).

On to the next steps:

![10-not-a-robot.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896264912/bj9kcbsbO.jpeg)

![11-setup-company-email.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896276537/uhJQXy_4C.jpeg)

![12-configure-gsuite.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896288540/uYQyw98U_.jpeg)

Now that I have proven that I'm not a robot (to a computer, aah, the irony). I'm ready to set up the email forwarding. First I need to verify that I do own the domain that I want to register. I can do this via a few different ways. I could upload a specific file to the domain or I could add a TXT or CNAME record to my DNS provider or I could add a meta tag to my websites landing page.

![14a-verify-domain.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896305767/kr3lhLi2j.jpeg)

![14b-other-methods.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896316814/CGZou_Ole.jpeg)

I've verified my ownership two ways, the first is with a tag on the homepage. All I had to do was to add the following meta tag in the <head> section of my homepage: `<name="google-site-verification" content="JDsToWsvqpcrHeTkR-1sdAJueNaSIezu6rf7iz1a-3o" />`.

The second verification is the DNS verification. Google detected that my DNS provider is [Cloudflare](https://www.cloudflare.com/) and provides [a specific page](https://support.google.com/a/answer/7174013?hl=en&ref_topic=4445219) to guide me trough the setup process. All I had to do was add a TXT record with content provided by Google.

I use Cloudflare to get free HTTPS on my website, so even if you surf to the HTTP endpoint, you will be redirected to the HTTPS endpoint and be secure. If you don't have HTTPS on your site already, Cloudflare or [Lets Encrypt](https://letsencrypt.org/) are options you should look into to get HTTPS on your site. Not only does it increase the credibility of your site, it will avoid future problems when [Chrome, FireFox and other browsers will show warnings that your site is not secure](https://www.pcworld.com/article/3161778/software/chrome-firefox-start-warning-users-when-websites-use-insecure-http-logins.html).

![15-open-control-panel.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896337917/iBn9-Be5Q.jpeg)

![16-add-mx-records.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896348953/LMRsmQsjY.jpeg)

So I go to the Cloudflare DNS setup for [morethancode.be](https://www.morethancode.be/) so I can add the necessary MX records. Above the DNS records, there is a simple setup that allows me to choose the MX record type, set the name to @, configure the specific server and priority and a time to live (TTL).

![17-cloudflare-dns-overview.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896380320/IawMLB__6.jpeg)

![18-cloudflare-add-mx-record.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896393315/rJUvf3FIl.jpeg)

Then click a few times next to verify that the domain actually belongs to me and the necessary MX records are in place. When all turns green, the setup has been completed.

![19-save-mx-records.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896410125/zOI_oM1rG.jpeg)

![20-verify-domain-setup.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896421371/s1l6GXj2X.jpeg)

![21-success.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896433813/Tx7vv2Mdi.jpeg)

The last part asks to set up a payment plan. I think I could skip this for the 2 week trial, but I want to use G Suite for a longer period of time, so I set it up right away. The biggest decision is to choose between the business plan (8€/10$ a month) or the basic plan (4€/5$ a month). There is an option for enterprises which does not have a monthly cost next to it (that never bodes well for my wallet) and since I don't need that volume, I ignored it. I took the basic option because that aligns best with what I need. I can always easily upgrade later.

![22-choose-plan](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380896922/Lr3cMmeGn.jpeg)

When I take the annual payment plan, I pay even slightly less than 4€/5$ a month. There is also the flexible payment plan which allows me to cancel my subscription month by month. Since I plan on running my business for a long time, I took the annual plan. After that I just need to fill in some billing information like my credit card and address. After that the suite is set up and I can receive email.

![23-choose-payment-option.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896463522/n-5S2x1CD.jpeg)

![24-billing-overview.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896477140/8xMZuBNLM.jpeg)

![25a-payment-info.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896493136/Wo-8LZq5r.jpeg)

![25b-payment-info.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896506528/AB8zRYybf.jpeg)

![25c-payment-info.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896518614/qOQd3F3WD.jpeg)

![26-congratulations-for-paying.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896530007/_rkME1xK0.jpeg)

Now that I have used G Suite for a few weeks, I'm very happy with the service. There's two annoyances, luckily it's nothing serious. When I want to upload a new profile picture, it shows an error message that it doesn't work. It worked once, but I haven't been able to change the picture since.

![error-message-when-changing-profile-picture.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896551984/08gWbvsGP.jpeg)

![error-message-when-changing-profile-picture-2.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617896563210/o23AJYMmX.jpeg)

The second annoyance is that I can't create a Google+ profile for my G Suite account. When I try to create, in the bottom left, I get a message that says that "Something went wrong." When that disappears, I'm left wondering what I can do and I haven't created a Google+ profile yet. It isn't a major issue, but I find it convenient when I send to a new email address and I get a clean name and picture instead of just an email address.

![something-went-wrong](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380898401/kzCOrGjeM.jpeg)

So this is how I set up my G Suite and my biggest annoyance. I hope that I've convinced you to try to use G Suite, because it's a very good service.

Disclaimer: I'm not being sponsored by Google, Cloudflare or 1Password. I promote these services because I think they offer the best services for a fair price.