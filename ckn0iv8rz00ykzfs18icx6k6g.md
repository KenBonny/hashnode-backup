---
title: "Internet Explorer JavaScript funkyness"
datePublished: Fri Apr 22 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iv8rz00ykzfs18icx6k6g
slug: internet-explorer-javascript-funkyness
tags: internet-explorer, javascript

---


In my work project, I needed to visualize date ranges. A quick Google search led me to the [vis.js](http://visjs.org/) framework. A easy to use framework that nicely renders the date ranges on a [timeline](http://visjs.org/examples/timeline/basicUsage.html). A bit of tweaking and it looked good... until I checked it out in Internet Exploder. It threw an exception within the framework that prevented it from rendering.

So the hunt for the solution began. I passed different objects to the Timeline constructor. I experimented with editing the framework where the exception occurred. I tried running the commands that were failing in the developer console to figure out what was wrong. I tried to put a new Timeline with a timeout to force the DOM to refresh. I tried all sorts of things.

It was all in vain. In the end, an update of the framework was what fixed the issue after I logged a bug. Their support was friendly and quick to fix the issue. The framework does a marvellous job at visualising Timeline data. Have not experimented with the other charts, but if they are as easy to use and well documented as the Timeline, it should be easy to set those up.

Moral of the story: keep your packages up to date for the best result.
