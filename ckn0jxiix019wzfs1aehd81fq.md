---
title: "Jimmy Bogards crossing the generics divide"
datePublished: Mon Mar 22 2021 13:45:09 GMT+0000 (Coordinated Universal Time)
cuid: ckn0jxiix019wzfs1aehd81fq
slug: jimmy-bogards-crossing-the-generics-divide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617382825483/GGvDIYC4s.jpeg
tags: programming, dependency-injection, dotnet, dotnetcore

---

 [Jimmy Bogard](https://jimmybogard.com/) wrote a blog quite recently about how to  [inject generic components without using the generic part](https://jimmybogard.com/crossing-the-generics-divide/). As I have struggled with this in the past, I thought I’d create a little implementation to check out how his article solves the problem.

So I created a GitHub repository named  [Crossing The Generics Divide](https://github.com/KenBonny/CrossingTheGenericsDivide). I did put it in a namespace KenBonny, not (only) because I did the copy pasting from Jimmy’s blog, but to show that this is not made by Jimmy. It’s just a tribute. I did do one thing different: Jimmy uses “validators”, I replaced that by “processor” as I wanted to return a result (which is just a bit of text to differentiate between the processors).

The most important lesson that I learned is that the composition has my preference, but that I cannot inject the general `IPolicyProcessor` (Jimmy’s `IPolicyValidator`) directly. I need to ask for the specific `PolicyProcessor<>` as I cannot pass the type of `Policy` to the factory method with basic dependency injection.

There is an example with the `IPolicyProcessor` that comes from a factory, but this required a bit of extra work in a console application. The `IServiceProvider` does not automatically get registered if I did it manually. To get this working, I had to set up a hosting environment (as described by the  [Microsoft docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0#call-services-from-main)). So if the normal execution is an asp.net project, this should work out of the box.