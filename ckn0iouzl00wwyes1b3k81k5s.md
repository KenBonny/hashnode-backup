---
title: "A Simple SpecFlow 3 setup in Rider"
datePublished: Mon Dec 30 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iouzl00wwyes1b3k81k5s
slug: a-simple-specflow-3-setup-in-rider
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617795957997/cv6TYtxiy.png
tags: unit-testing, ides, dotnetcore

---

> Checkout the [Rider plugin for SpecFlow](https://plugins.jetbrains.com/plugin/15957-specflow-for-rider) for an easier integration!

Recently, I've gotten some mails on how to use [SpecFlow 3](https://specflow.org/) with [Jetbrains Rider 2019.3.1](https://www.jetbrains.com/rider/). So I thought I'd update my how-to posts with this addition. If you are just here for the code, you can find it on my [GitHub account](https://github.com/KenBonny/SpecFlowSetup/).

I'm keeping this specifically simple and small. As soon as the project does what it needs to, I'll stop, because my previous articles on this explain the rest (which still works) in more detail.

![Create new project in Rider](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380663153/umsPCqodT.png)

I'm just creating a new solution from Rider and adding a _Unit Test Project_ with the xUnit framework. If you're unaware, [I'm a big fan of the xUnit framework](https://kenbonny.net/tag/better-code-with-tests/). Then I added 3 Nuget packages:

1. SpecFlow
2. SpecFlow.Tools.MsBuild.Generation
3. SpecFlow.xUnit

At the time of writing this article, all these packages have version 3.1.74.

When I want to install the SpecFlow.xUnit package, Nuget complains that [xUnit](https://xunit.net/) is not the latest version and I have to update from 2.4.0 to 2.4.1. I took the opportunity to use Riders awesome _Upgrade All Packages In Solution_. Quick and easy.

Now that everything is installed, I just insert a File and name it `SomeTest.feature`. I type away and create a simple SpecFlow file. I could now build and run the test, but out of habbit, I create a `SomeTest.steps.cs` file and put in the basic `Binding` and `Scope` attributes above the class.

Then I run the tests and check the output.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380664727/Z2lvjBLSq.png)

As you can see on the right part of the output pane, it still works as in my earlier article and I can just copy out the relevant parts for my test setup. After I flesh out the test a bit, I run it again and get a green check.

I did exclude the generated `*.feature.cs` as it contains an unknown element, but does compile. The error does show the annoying red line below the file hierarchy it influences. They are not needed in the project (or the git repo, I excluded them via the `.gitignore`) to make the tests run. They are being included during the build step anyway.

SpecFlow 3 works very nicely with DotNet Core 3.x and the Rider IDE. It's a great time to be a (DotNet) coder!
