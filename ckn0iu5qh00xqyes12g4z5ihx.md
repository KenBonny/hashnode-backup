---
title: "Good names are immensely important"
datePublished: Wed Feb 01 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iu5qh00xqyes12g4z5ihx
slug: good-names-are-immensely-important
tags: programming, general, programming-tips

---


A good name can be the difference between explaining what a particular part of code does and confuse you. Names also tell you a lot about the quality of the codebase.

In code, names convey meaning. It is the most important reason to choose a good name. A name should describe what a class, function, method or delegate does or it should describe what a variable, field or property contains. While writing code, using good, descriptive names can be easily overlooked because when I'm busy with writing actual code, I choose the first name that I find appropriate. In most scenarios that is not the best name. Good names aren't useful for the compiler, it doesn't care. They are for the next developer who needs to be in the code to make a change, do maintenance or solve a bug. When I'm reading code, it becomes abundantly clear when good names have been used or not. They make reading code a breeze. Refactoring code to use good, descriptive names shouldn't be skipped. The longer I wait, the harder it is to find the original meaning and choose a good name.

The problem with choosing a good name is that it's difficult. A good name isn't just descriptive, it avoids confusing the reader and removes the need for mental mapping. I shouldn't spend time deciphering what a confusing or ambiguous term means. A good name should use domain language so the whole team understands what is happening.

Good names aren't just for classes and methods only, the whole codebase should be full of them. Private methods, fields and variable names should all have good names. This allows code to be easily read and understood because all levels of the code describe what is happening. If I can easily understand what is going on, then I can easily determine what is wrong and fix it. When I change something, it's my responsibility to check if all names are still correct so the names stay up to date and don't start to be confusing.

Let's take a look at a bad name I recently encountered and how I would improve them. (I couldn't touch that part of the codebase at that time.) The code I was working on needed to filter results and filters needed to be added or removed easily. There is a `Filter` base class with multiple implementations. This is for a warehouse management system. I encountered a `FilterValid`. The first problem is that I have no idea at first sight what the properties of a valid record are. So, I checked the implementation and found that it did 3 things:

1. It checks that the property `IsEnabled` is set to `true`;
2. It checks that the property `IsError` is set to `false`;
3. And it checks that a place in the warehouse is not occupied.

Besides a confusing name, it violates the Single Responsibility Principle. This is related to the confusing name: if it checks for validity, why does it check for occupation? The second problem I noticed is that there is another filter with the name `FilterOccupied`. This does what functionality 3 asks for, so it's duplication of code.

What I would do is split the `FilterValid` up in two filters: `FilterIsEnabled` and `FilterNotInError`. I think it's fairly obvious what each filter should check. That is just the kind of easily readable code I want in a codebase. I would use these two filters, together with the already existing `FilterOccupied`, to check all required conditions.

Another bad name I encountered was in the line `if (!Apply())` then it did something. Now when I read `Apply()` I would think it would do something, not check for validity. What the (private) method does is determine whether the filter should be used based on search criteria. A simple rename to `ShouldApplyFilter()` or `ShouldExecute()` would let me know what the function does and I don't need to investigate.

These are not the only problems I'm finding in the codebase related to names, but they are good examples. All those little infractions make me trust the code less and less. If I encounter a descriptive name, I doubt that it does what it says it does because the rest of the code isn't as trustworthy. This means I lose time while investigating bugs or making changes. Use good names, it will make your development a lot easier.
