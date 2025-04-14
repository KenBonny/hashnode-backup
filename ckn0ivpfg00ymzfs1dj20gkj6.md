---
title: "Marvel Movie Marathon"
datePublished: Mon Sep 14 2015 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ivpfg00ymzfs1dj20gkj6
slug: marvel-movie-marathon
tags: css, github, web-development

---


Last week (or maybe a little bit longer ago), I came across an image of a [marvel movie marathon](https://www.google.be/search?tbm=isch&q=marvel+movie+order) and in what order you should watch the movies and TV series so you'd view them in a chronological order. To simplify things, I made the site [marvelmoviemarathon.com](http://marvelmoviemarathon.com/). Just to be clear, let me reiterate: I'm not affiliated with Marvel in any way and if I'm in violation of certain copyright laws, [contact me](http://marvelmoviemarathon.com/#contact) and we'll work something out.

But I wanted to document how I host it on github and which resources I use: bootstrap, a bootstrap timeline, jQuery and mustache.js. (You know, promotional material and documentation for posterity.)

## Hosting

Since this is nothing more than a static page that reads a json and parses it, it shouldn't be hard to host it. I recently heard about [GitHub Pages](https://pages.github.com/) and wanted to try it out. It's very easy. You create a branch on your project named "gh-pages" and only check in the code you need to run a website. The default page is index.html (I think default.html will work too, but I haven't tried this). Besides that, you need to check in all files on this branch that your site needs (such as javascript and css files). If you're writing a framework and you want to host a simple (or even an Angular site or a [blog](https://help.github.com/articles/using-jekyll-with-pages/)) for the project, you can very easily do it this way.

Once the necessary files are checked in, you can check your site at <accountname>.github.io/<repository>, so you can also check the site at [kenbonny.github.io/MarvelMovieMarathon](https://kenbonny.github.io/MarvelMovieMarathon/).

The last step was to get a domain to serve pages from the github.io address. You go to the dashboard on your DNS host and create an ALIAS that points to <accountname>.github.io. To let github know to which project you want a domain to redirect, you need to check in one more file to the "gh-pages" branch of your project. This file has to be named "CNAME" with the address of the site as the sole content (in this case "marvelmoviemarathon.com"). The capitalization is very important! If you don't use capital letters, github won't recognize it as the file that contains the domain.

There you go, now you know how I cheaply host [marvelmoviemarathon.com](http://marvelmoviemarathon.com/) on [GitHub Pages](https://pages.github.com/).

## Technical overview

The main point was to keep this site simple, so updating it with new content would be very easy. The reason I wanted to talk about hosting first is because you'd know I don't have a server side component or database that provides the data, it's just a data file in json format. So if I need to add or edit records, I only have to update one file and I'm done.

The first step is to load the data via an ajax call using the [jQuery](https://jquery.com/) api (this also helps to keep the site quite responsive). When I have the data, I load two templates I build with [mustache.js](https://mustache.github.io/). Why two templates? One for the left and one for the right side of the timeline. Filling in the correct class won't change the position of the poster in the timeline. I wasn't planning on making those changes in code, it was a lot easier to create two templates. Sue me, I'm a lazy programmer.

The code of the timeline isn't entirely mine either. I found the basis on [codepen](http://codepen.io/betdream/pen/Ifvbi), which had one slight problem: it didn't adjust nicely to mobile devices. A tablet will work decently, but when viewed on small mobile devices, it would keep the middle divider line. This would make the timeline items almost unreadable and (more important) very ugly. So [I modified the css](https://github.com/KenBonny/MarvelMovieMarathon/blob/gh-pages/css/timeline.css) with a media query that will place the divider line to the left when the screen has a smaller width than 450px and place all the timeline items to the right. With the two different templates, the posters alternate between being displayed left and right, which keeps the visual flow from the timeline.

A big thanks to [Bootswatch](https://bootswatch.com/) for providing the [Journal theme](https://bootswatch.com/journal/).

## Summary

The most important part is to remember that hosting a small site on [GitHub Pages](https://pages.github.com/) is very easy and cheap with a pretty reliable uptime, but no guarantees. The rest of the article is just to make me look important. :)
