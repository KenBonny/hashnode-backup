---
title: "SpecFlow steps generation and general Rider changes"
datePublished: Mon Jul 23 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j0f6b00ztyes1aox95ryf
slug: specflow-steps-generation-and-general-rider-changes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381206400/eXKAGvgAJ.jpeg
tags: unit-testing, unit-tests, ides

---


In May of this year, I wrote an article on how to get [JetBrains Rider](https://www.jetbrains.com/rider/) to generate [SpecFlow files](http://kenbonny.net/2018/05/28/generating-specflow-files-in-rider/). The biggest problem I still had back then was that I couldn't generate the step definitions. I finally found a workaround so I don't need to use Visual Studio anymore.

After creating a test project with the full dotnet framework, I add the SpecFlow and SpecFlow.xUnit packages. I insert a _.feature_ file and write a test. The save generates the _.feature.cs_ file (see [my previous article](http://kenbonny.net/2018/05/28/generating-specflow-files-in-rider/) how to set that up).

Now that I can execute the test, I just run it.

![generate-steps](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381199121/6Vk8jNaLf.jpeg)

In the test output window is the basic setup for the current steps. Just copy and paste the steps into a new file and enjoy the steps generation. When I write a new Given, When or Then step, I just rerun the test and check the output for the method signature.

![additional-step-generation](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381200720/6tsXHh8Ec.jpeg)

Have fun generating feature steps in Rider.

> While setting up a small test environment, I found out [SpecFlow](https://specflow.org/) isn't dotnet core compliant yet, but they're hard at work to make that happen. Keep up the good work guys!

> While installing the SpecFlow packages, I found out the default dotnet core packages location changed from the solution folder to the _C:\\Users\\\[UserName\]\\.nuget\\packages_ folder.

> In Rider, there is a nice feature to nest files. In the _Solution_ window, there is a button to open the _File Nesting_ window. Just add a _Parent file suffix_ with ".feature" and a _Child file suffix_ of ".feature.cs; .steps.cs".
> 
> ![nested-files](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381202204/AqTEcCOCP.jpeg)![file-nesting](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381203627/sc1KAan1q.jpeg)
> 
> Now feature files will nicely be nested.
> 
> ![feature-nesting](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381205117/u8IneVApr.jpeg)

Hopefully these tips and tricks will prove useful to some Rider users out there.
