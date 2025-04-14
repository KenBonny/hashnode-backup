---
title: "Specflow GenerateFeatureFileCodeBehindTask error"
datePublished: Mon Aug 10 2020 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j06kl010nzfs17y08dek3
slug: specflow-generatefeaturefilecodebehindtask-error
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617460582842/uD5pmLUZO.png
tags: csharp, unit-testing, programing, dotnetcore, cucumber

---


In the past few months I have been contacted by a number of readers who notified me that the code for my [A simple SpecFlow 3 setup in Rider](https://kenbonny.net/2019/12/30/a-simple-specflow-3-setup-in-rider/) blog post was broken. I took me a while, but I've finally taken the time to fix it.

Some time ago I already looked at it and I saw that with the upgrade to dotnet core 3 there was a breaking change with SpecFlow `.feature.cs` file generation. Due to a change in the MSBuild pipeline, the files were not generating correctly. The following error was shown during the build of the solution.

> The "GenerateFeatureFileCodeBehindTask" task failed unexpectedly.
> 
> MSBuild

Fortunately, upgrading to the latest version of all the packages fixed all the errors. Thank you SpecFlow team!

The [SpecFlowSetup project on GitHub](https://github.com/KenBonny/SpecFlowSetup) has been updated and should work again.
