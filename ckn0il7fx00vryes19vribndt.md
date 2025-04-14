---
title: "Adding a CSP to my site"
datePublished: Mon Mar 26 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0il7fx00vryes19vribndt
slug: adding-a-csp-to-my-site
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380496175/3rSIob19f.jpeg
tags: security

---


A few weeks ago [Troy Hunt](https://www.troyhunt.com/) and [Scott Helme](https://scotthelme.co.uk/) released [Report URI JS](https://scotthelme.co.uk/launching-report-uri-js/). I wanted to test it out to see how easy it was to implement. The library is very easy to use, the hard part is still setting up the CSP.

My company's site [morethancode.be](https://www.morethancode.be/) was not yet protected by a [CSP](https://scotthelme.co.uk/content-security-policy-an-introduction/). I had not taken the time to implement it yet since I could not add a report endpoint with the html meta tag. Having a CSP without a way to report on errors is asking for problems. This new library changed that and I finally took the time to implement a CSP.

Crafting a CSP from hand is isn't the easiest thing to do, it's easy to overlook something. That's why there are tools to automate this process. The one I used is a [Fiddler](https://www.telerik.com/fiddler) extension to [collect CSP rules](https://github.com/david-risney/CSP-Fiddler-Extension). I start up Fiddler after I've installed the extension and I make sure the "Enable Rule Collection" option is checked on the "CSP Rule Collector" tab.

![2-enable-csp-rule-collection](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380490503/PiS1dIPM4.jpeg)

To get the CSP rules, all I have to do is visit the site I want to profile while Fiddler is running and the collector will do the rest. I can get the contents by selecting the request to the page and in the window to the right the policy appears. With a simple right click I have the option to copy the policy.

![3-csp-rule-collected.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380491861/2bQthInCy.jpeg)

To get the CSP into my site, I add a meta tag with the CSP in it. I do have to remove everything before `default-src` as that is where the policy actually starts. I have placed the CSP at the beginning of my html file, because the policy only applies to tags that appear after the CSP meta tag.

_I'm sorry all my code examples are without smaller and greater brackets, but Wordpress screws up how to display those characters and makes them part of the page. I would effectively alter the HTML of this page and I'm not taking chances what Wordpress will do with this._

```
meta http-equiv="Content-Security-Policy" content="default-src 'none';
 font-src https://cdnjs.cloudflare.com https://fonts.gstatic.com;
 img-src 'self' https://www.google-analytics.com;
 script-src 'self' https://cdnjs.cloudflare.com https://www.google-analytics.com https://cdn.report-uri.com;
 style-src 'self' https://cdnjs.cloudflare.com https://fonts.googleapis.com;
 connect-src https://morethancode.report-uri.com;
 upgrade-insecure-requests;
 require-sri-for script style;"
```

This is where the fun starts because now I get all kinds of errors. Time to start fixing them. The first one on the list is to replace all normal script and style tags and replace them with tags that include SRI verification. To get the CSP [to verify that all SRI's are in place](https://scotthelme.co.uk/enforcing-the-use-of-sri/), I do need to check that the component `require-sri-for script style;` is in the CSP. It isn't included in the generated CSP.

Unfortunately, at the moment, there seems to be a [bug in Chrome](https://security.stackexchange.com/questions/180450/why-does-chrome-tell-me-that-the-csp-require-sri-for-directive-is-only-impleme) that ignores this rule. A Chrome flag prevents it from being executed. The flag however cannot be found in the `chrome://flags` settings. (Just to be sure, be careful what you change in those flags settings.)

SRI stands for [SubResource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) and it's been discussed in [great](https://www.troyhunt.com/tag/sri/) [detail](https://scotthelme.co.uk/tag/sri/) by people who know much more about it. If you don't know what it is, go check out some those posts first.

Ok, now that we all know what SRI is, how do I get those tags? I get my resources from [Cloudflares CDN](https://cdnjs.com/), they make it very easy to include SRI information. I just find the package I want to include, I go the details page. Next  to the line that indicates which script or style I want, there is a copy button. The copy button has a drop down menu with several options and one is "_Copy Link Tag with SRI_". That gives you the whole tag, including the SRI hash.

![0-cloudflare-sri](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380493279/_OL5q-WGa.jpeg)

Now I just select the tag with the script or style and paste over the new tag. That's all I need to do to include SRI on my Cloudflare CDN links. I wish Cloudflare would update the default copy behaviour to include SRI information. Even if I don't do anything with it, I then have it in place for when I want to enable CSP and SRI.

```
without SRI
link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.7/css/bootstrap.min.css"
script src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.7/js/bootstrap.min.js"
with SRI
link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha256-916EbMg70RQy9LHiGkXzG8hSg9EdNy97GazNG/aiY1w=" crossorigin="anonymous"
script src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha256-U5ZEeKfGNOja007MMD3YBI0A3OSZOQbeG6z2f2Y0hu8=" crossorigin="anonymous"
```

My own scripts require some additional work. Since I can't just get a hash for the `integrity` attribute on the html tag from Cloudflare, I need to calculate this myself. There are a lot of good sites that do that for me, but I like the one on [report-uri](https://report-uri.com/home/sri_hash) best. I paste in the link to the style or script and I get the full meta tag including hashes.

![1-report-uri-hash](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380494731/0XAlNr3Ek.jpeg)

There is one last script that needs to be taken care of: the Google analytics script embedded in the page. There are three ways to deal with embedded scripts:

- Adding a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce), but since I cannot run anything on the server since this is served from [Github Pages](https://pages.github.com/), this option is out for me.
- Add a hash of the script to the CSP between single quotes in the \`\`\`script-src\`\`\`. The easiest way to find the hash is to open the page in Google Chrome and look at the error in the developer console. The expected hash will be displayed in the error.
- Move the script to a separate script file and add a script tag with an integrity hash.

An additional script is the easiest and cleanest way to get the CSP to comply. It's uniform with the other scripts and I do not need to add a hash to the CSP. If I have a few inline scripts, I won't know which hash to update after I change or add a script. So I can save my future self a lot of headaches by moving the script to another file.

As a last remark, I notice that all the `'self'` references are at the end of each component. In every example I find, they are placed at the start of each component. Once I moved those, I notice that the images served from my domain display correctly on my site.

Next week, I'll write about reporting of the violations. In hindsight that was probably something I should've done earlier. So stay tuned to know how I set up report-uri to collect all my csp reports.

**Update**: Twitter user @spazef0rze pointed out that the "bug" in Chrome is an experimental feature. He pointed out that I can activate it under: [chrome://flags/#**enable**\-experimental-web-platform-features](//flags/#enable-experimental-web-platform-features)

%[https://twitter.com/spazef0rze/status/978990058169487360]

[Header image source](https://www.keycdn.com/support/content-security-policy/)
