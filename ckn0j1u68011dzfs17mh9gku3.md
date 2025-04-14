---
title: "Theme change"
datePublished: Mon Oct 02 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j1u68011dzfs17mh9gku3
slug: theme-change
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381272281/Tg2vfOP1S.jpeg

---


For a few weeks now, I've updated the look and feel of this blog. Let me give you a quick explanation why I did this.

At a certain point, I noticed that the https indicator in chrome turned to grey instead of the expected green padlock:

![bad-example](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381269537/gQ_xYVGy0.jpeg)

As I think it is important that your browser tells you that my site is securely presented to you, I immediately began to search why this occurred. The answer was pretty simple: a search field was posting to an http endpoint. The debug console of the chrome developer tools told me this and I confirmed it with a quick search through the page source.

Since version 62, chrome (not sure about other browsers) will flag all sites that [post any data to http endpoints as insecure](https://blog.chromium.org/2017/04/next-steps-toward-more-connection.html). I know firefox, just like chrome, flags sites that post passwords over http as insecure, but chrome takes it one step further. If this has changed in other browsers, then I'm not aware of it.

Now that I know what the problem is, the obvious solution for me was to change the theme because that will change the form action as well. So I started looking for a theme that I like and in which my site would load securely.

After some digging, I found this theme and I activated it. Unfortunately, I did not think it through. When I viewed the preview, the site is rendered on Wordpress and it generates the form action with a https url. So all looks good in preview, but when I updated the theme, chrome still displayed the grey https warning.

Wordpress apparently generates a site loaded via a custom domain like [kenbonny.net](https://kenbonny.net) with http form actions. Not sure why, but it does it. So I was back at square one.

I started digging into the Wordpress admin console and looked for things I could tune. On the theme customization page, there are widgets available. In the sidebar and footer sections, there were search widgets present and I found no way to tell them to post to a https endpoint instead of an http one.

As a final solution, I removed those widgets. For the time being, search functionality is disabled in favour of having the green padlock in the browser url bar. As a side effect of me acting to fast, I now have a new theme. At least my green padlock is back.

![good-example](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381270757/Qq_S0ezZO.jpeg)

P.S. If anybody knows how to make Wordpress post search forms to an https endpoint, please [get in contact](http://kenbonny.net/about/). I would appreciate it very much.
