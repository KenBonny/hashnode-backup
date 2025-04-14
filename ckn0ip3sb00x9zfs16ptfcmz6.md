---
title: "Adding reporting in the mix"
datePublished: Mon Apr 02 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ip3sb00x9zfs16ptfcmz6
slug: adding-reporting-in-the-mix
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380677901/R948aSuZS.jpeg
tags: security

---


Last week, I wrote about [crafting a CSP](http://kenbonny.net/2018/03/26/adding-a-csp-to-my-site/) for my professional site. This week, I'm going to add reporting so that when things go wrong, I know what went wrong.

The last thing I did when implementing a CSP should have been the first thing I did, which is to add reporting. I can find all errors in the developer tools from my browser. Adding reporting consists of three parts: setting up a report-uri.com account, adding the report uri js script and adding configuration in my site.

The script is available from the [Report-URI CDN](https://scotthelme.co.uk/launching-report-uri-js/). This is available with SRI hash so it's just a copy and paste next to all the other scripts. The configuration part is a `script` tag with the `type` attribute set to "text/json" and the `id` attribute set to "csp-report-uri".

There is already a version 1.0.1 out, so I have to make sure I use the correct version. I initially used version 1.0.0 and there was a problem with sending reports to the service. That seems to be fixed now. This could also be connected to the issue with Chrome that I wrote about [in the previous post](http://kenbonny.net/2018/03/26/adding-a-csp-to-my-site/).

```
{
  "keys" : ["blockedURI", "columnNumber", "disposition", "documentURI",
    "effectiveDirective", "lineNumber", "originalPolicy", "referrer",
    "sample", "sourceFile", "statusCode", "violatedDirective"],
  "reportUri" : "https://{subdomain}.report-uri.com/r/d/csp/enforce"
}
```

Now, there is a part missing in the "reportUri" url. The "{subdomain}" needs to be replaced. This has to come from my Report URI account. So, I head over to [report-uri.com](https://report-uri.com/) and I register. It's a very straightforward process, all I need is an email and a strong password.

When I first logged in, I noticed this big banner that shows me the steps to a good and secure account. It's a nice way of letting me know they want me to succeed, have a secure account and at the same time make their service easy to set up. I love that banner since it saved me from possible mistakes. A simple guide to getting started.

![7-report-uri-first-login.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380672262/YsypfeGRu.jpeg)

After a few minutes, I receive a mail to verify my account. So that takes care of the first check. The customise part takes me to a setup page where I can replace the random text to get a more personalised report uri. That is also the part I have to add in the data script from earlier. The configuration tab should not be skipped, because only domains that are whitelisted there will be processed. This is great because a random domain cannot use my report-uri and flood my logs. The last step urges me to enable 2FA. I highly recommend this as it makes your profile so much more secure. If you don't know how to do this, I recommend 1Password to [store both the password and the 2FA codes](http://kenbonny.net/2018/03/05/two-factor-authentication-via-1password/). If you don't have 1Password, I recommend [Authy](https://authy.com/) as a backup plan.

Now that I have a report uri endpoint set up, I can rest easily. Unfortunately reporting will not work yet. The outgoing endpoint "subdomain.report-uri.com" needs to be whitelisted in the CSP under the `connect-src` section. I first thought I needed to add this to the `default-src` section. I did not know that this contradicts the 'none' setting.

If I don't add the report-uri address to the CSP, it tries to send a report to the reporting endpoint, which is not allowed and generates an error. That error then needs to be sent to the reporting uri, which generates an error. This keeps going and going and causes serious slowdowns on my development machine. In just a few seconds, this was the error count in my Chrome developer console. The reports that are generated also don't get sent to the report uri as the CSP blocks this because it thinks it's a forbidden endpoint.

![4-massive-errors](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380673768/qBT2W5hO4.jpeg)![5-error](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380675106/ioqFMcBKh.jpeg)![6-error-details](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380676505/Ik_RgrL03.jpeg)

Now that I've secured my entirely static site (which is kind of overkill, but it gave me an excuse to experiment with CSP), let's talk about the down sides. I've noticed that my personal scripts need a new hash each time I check in a new version of the `index.html` page. Even if those scripts did not change. The strange part is that the second commit with the new hashes does not cause a problem. So, every change to `index.html` is followed by a commit with the new hashes for my style and my two scripts. The CDN scripts are fine. I'm not sure why this is happening or how to fix this, if somebody knows, [get in touch](http://kenbonny.net/contact/) and let me know.

Since I wrote the previous paragraph last week, I have updated my index page and seen that the hashes of my personal script and style don't need to be updated. I have disabled caching on [Cloudflare](https://www.cloudflare.com/) for js and css files. This seems to have fixed my problem with incorrect hashes. I'm not sure why I get errors from the cached files, but they're not an exact match. I think the cached files have different metadata (such as the file created date) that changes the hash. I can't confirm this though, as the errors are pretty random.

There is another way to get around this: I could replace the hash in the `integrity` attribute with a placeholder. Then a Cloudflare worker could calculate the hash on the fly and replace the placeholder with the appropriate hash. For now, my professional focus is on the Belgian market so I'm not overly bothered that other parts of the world are not getting the best experience. I am still looking into this, so if you know something that could fix this, [get in touch](http://kenbonny.net/contact/).

Now that I have a CSP set up and reporting going, I'm pleased that this is live. I do recommend to do this in stages for larger sites and to start out as report only. CSP can wreck your site very fast when a stylesheet or javascript isn't loaded. Doing this right does bring a lot of security benefits. Especially for public facing sites that allow user input. That's a topic for another blog post though.

In closing, I want to thank Scott Helme (and Troy Hunt) for making such an easy to use [reporting service](https://report-uri.com/). I asked Scott a question and got an answer within 24 hours. Top notch service.

 

%[https://twitter.com/Scott_Helme/status/977275969722560512]

[Header image source](https://www.keycdn.com/support/content-security-policy/)
