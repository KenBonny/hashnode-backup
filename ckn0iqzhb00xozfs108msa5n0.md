---
title: "CloudFlare in front of a hosted WordPress blog"
datePublished: Tue Aug 16 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iqzhb00xozfs108msa5n0
slug: cloudflare-in-front-of-a-hosted-wordpress-blog
tags: cloudflare, blog, wordpress

---


This blog is hosted on [WordPress](https://wordpress.com/) and it would cost me money if I would buy a certificate through WordPress. To add HTTPS to my blog, I route it through the free service [CloudFlare](https://www.cloudflare.com/). I never found a decent blog post how to do it, so I'll detail what I know in this post.

The process is super easy: log into CloudFlare and begin a scan to add a new site. On the DNS records page, delete the two A records. This will cause a circular reference otherwise. Instead add a CNAME record for your site to lb.wordpress.com. The final step is to choose the free tier so it won't cost you a cent.

The only drawback is that WordPress can't identify the IP address of the visitor anymore. So the stats on the countries can't be determined by WordPress anymore. Those numbers can be found on CloudFlare. Installing the [CloudFlare plugin](https://wordpress.org/plugins/cloudflare/) can solve that problem, but on a free blog, installing plugins isn't allowed.

That's it. Enjoy your site with HTTPS.
