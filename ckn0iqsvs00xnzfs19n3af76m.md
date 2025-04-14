---
title: "Clean code is like a clean desk"
datePublished: Mon Jul 31 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iqsvs00xnzfs19n3af76m
slug: clean-code-is-like-a-clean-desk
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380757080/FqEsTH2NB.jpeg
tags: programming, programming-tips

---


A few weeks ago I was discussing this with colleagues and I saw what they meant, but it didn't click until I discussed this with my wife.

When I discussed this with a few colleagues, they referred to a clean bedroom. Everything has a place. If I need something, I know where to look for it. When I need to place something new in, I know in which part of the room I need to look. Clothes go into the closet, papers go on the desk, etc.

I understood what they were saying, but not all the puzzle pieces fell into place. I didn't have the aha-moment. I had that moment when I was talking to my wife about her first day at her job after her last holiday. My wife is very organised, so she was talking about her desk being spotless when she left. All paperwork was done, the ongoing dossiers were handed over to a colleague. The pending dossiers were put on hold for 2 weeks and filed in one of her drawers so they couldn't get lost.

When she got back, her desk looked like it had seen combat. Papers were thrown on her desk like it was an afterthought. She couldn't find anything when coworkers started asking her questions about new dossiers. It's only when she organised everything into different piles (immediate and longer term, inbound and outbound, etc.) and started working through them one by one, that she got control over her desk again. It took her an hour or two to organise, but it made her job easier, she got through dossiers faster, she found papers quicker.

This is when I clearly saw the similarities with my job. I don't have stacks of paper on my desk, but I do have stacks of source code files. When there are different concerns in a single source code file, I'm not sure if I should split the new functionality into a new file or just add it to the pile. Just kidding, I would take the opportunity to clean the code. That has the consequence that the feature will take more time, because I'm not focusing on the new feature alone. I'm also focusing on cleaning the code.

This brings us to the next problem that a codebase isn't cleaned as easily as a desk. But there are similarities here as well. On a desk, there is an automatic separation of concerns. The papers contain the data that will need to be processed, the desk is infrastructure. In code it's a lot easier to get those concerns mixed up. Where does a scheduler live? Is it part of the domain or is it infrastructure? What about an XML parser? Where does infrastructure end and domain logic begin?

Just because it will take more time, doesn't mean it's a waste of time. The resulting code will be easier to understand. Separation between domain logic, infrastructure code and tests will appear through the projects and file structure. Adding new features and fixing bugs will take less time. The location for new features will jump out or will take less mental effort to locate. Bugs tend to hide in places where it is hard to reason about the logic of the code. So when code is clear and understandable, bugs will stand out.

 

Different perspectives can provide new insights into why I do things I take for granted. This was a nice reminder why I should have a clean workplace... I mean code base. Finally, remember that there is a place for everything and put everything in its place.
