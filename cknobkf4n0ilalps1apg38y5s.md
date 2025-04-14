---
title: "Moving my blog to Hashnode"
datePublished: Mon Apr 19 2021 08:11:32 GMT+0000 (Coordinated Universal Time)
cuid: cknobkf4n0ilalps1apg38y5s
slug: moving-my-blog-to-hashnode
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618815817590/jfIFvraz1.jpeg
tags: cloudflare, blog, wordpress, hashnode

---

Lately I've not been very happy with WordPress as the home for my blog. It has a ton of features I don't need, it has hundreds of free premade templates of which I chose the least ugly one and it is quite complex. Each time I visited my own blog (not something I do every week or even month), I got reminded that WordPress tracks my visitors.

![tracking in brave and chrome.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618815478465/7fBdqFJBU.png)

So I set out to find an alternative. At first, I thought about creating my own platform. Something like Gatsby or Hugo. After looking into that for a bit, I found it too much work for a blog with my own personal thoughts. I don't want to get bogged down into maintaining a blog platform, making sure the certificates are up to date, improving speed and efficiency, getting the RSS feed right, adding support for twitter and code snippets, etc. I just want to log in, write an article with some content and publish it. I admire the guys who have the time and dedication to maintain their own platform, but I know I have too much going on in my life to find or make time for that commitment.

After some googling and looking around, I stumbled upon  [Hashnode](https://hashnode.com) . It is simple but not limiting, I can write in markdown (which I personally like) and it has just the right kind of configurability for me. It supports RSS feeds, has a build in night mode (I don't need it personally, but I know a lot of people like it) and it's a hosted solution (meaning, I log in, write content and publish). It even offers custom domains for free!

While I was moving my ~130 articles from WordPress to Hashnode, which will get a more thorough technical post in the future, I noticed a number of nice little benefits. My number one: Hashnode does not track the readers.

![no tracking in hashnode.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618815524630/F7GvEROkt.png)

A curated list of tags is another one I got used to quite quickly. This forces me to choose from the available tags, but it ensures everybody uses the same tags to make posts with relevant topics easy to find. In WordPress, I could add any tag I wanted, but it also meant the community is pretty scattered.

The last benefit, is that I can take backups to a GitHub repository. Right now, I would need to edit and save each post individually for it to be backed up, but there is a feature coming to back up the legacy posts. It quite literally backs the markdown up to a repository that I control. If I ever need or want to switch blogging platform, I've got my content in an easy to access format under my control. As I mentioned earlier, getting my content out of WordPress and into Hashnode was not a simple task and I like that Hashnode makes this a lot simpler. It says to me: you are free to leave if you do not like our platform. This implicitly means that they have to step up and provide a great platform. This really resonates with the way I like to work.

It's not all sunshine and happiness. The main thing I'm going to miss is the feature to post blog (or stories as they are called in Hashnode) at a later date. This means that I cannot finish writing a blog post and schedule it to release on Monday at 10 o'clock. I hope they add this quite fast as I'm fond of this feature.

The second and more problematic thing, is that Hashnode does not support the url format that WordPress uses: `[domain]/[year]/[month]/[day]/[slug]`. the `[year]/[month]/[day]` part is not added by Hashnode and I've had a bit of trouble to work around that. A big thanks to  [Cloudflare](https://www.cloudflare.com) for supplying such an awesome platform that made this quite easy to work around. The details of this will come in the follow up post with more details.

The last thing that kept me on the fence for a little while, is the speed. Although the DOM draw happens quite fast, the Hashnode blog is heavier and takes overall longer to load. Because I like the writing experience a lot better and they don't track users, I am going through with the switch. Also, ignore my awesome editing skills. The speeds are taken by loading my lastest blog post  [Jimmy Bogards crossing the generics divide](https://kenbonny.net/2021/03/22/jimmy-bogards-crossing-the-generics-divide) from both the WordPress and the Hashnode site.

![speed comparison.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618815576949/YfAIMIsHw.png)

Overall, I'm quite happy with how my Hashnode blog looks, shows code and handles Twitter and other references. That is why I'll be moving my blog over to the new platform next weekend. If you want to keep receiving updates whenever I post them, make sure to check that that still works after next week. I've made sure the impact is as small as possible, but I'm sure some issues are bound to pop up. Let me know when they happen and I'll fix them as soon as I can.