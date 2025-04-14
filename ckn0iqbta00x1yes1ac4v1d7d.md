---
title: "Clean Architecture Applied"
datePublished: Mon Feb 18 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iqbta00x1yes1ac4v1d7d
slug: clean-architecture-applied-intro
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380735308/WEkr8Hcs_.jpeg
tags: csharp, software-architecture, dotnetcore

---


To make my life easier, I automated the generation of the invoices that I send as an independent contractor. This gave me the perfect excuse to build something that follows the rules set out by [Uncle Bobs Clean Architecture](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). At least how I interpret them. Use this blog post as a source of inspiration on how to do it... Or a cautionary tale, however you see fit.

### What is this Clean Architecture?

Let me start with defining what Clean Architecture is. It is a set of guidelines that will help me structure a system so it becomes a collection of loosely coupled components with the most stable parts, hopefully the domain, at its core. The further I move from the core, the more volatile the components become. Where volatile means "changing a lot" and not "behaving in unexpected ways".

Just as the SOLID principles are there to provide guidelines to write maintainable code, so is Clean Architecture a set of guidelines to write easily maintainable and extendable software. As these are guidelines, I can follow or abandon them as I see fit. One of the founding rules for me, is that pragmatism takes priority over blindly following rules. The latter leads to religiously following a doctrine, which in itself is a dangerous practice. Always think for yourself and check that the pros outweigh the cons.

![Clean Architecture diagram](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380733632/KD6kGJprL.jpeg)

Source: [http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)  

The image above resembles the [Ports and Adapters](http://www.dossier-andreas.net/software_architecture/ports_and_adapters.html) and the [Onion Architecture](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/). That's because Clean Architecture uses a lot of the same principles. It borrows heavily from Ports and Adapters for interoperability and how different components work together. It also uses the layered approach that can be found in the Onion Architecture. I think it kind of uses the best of those two systems and sprinkles a bit of best practices and pragmatism on top.

For the sake of the experiment, lets apply these guidelines. I will not go further into the specifics of Clean Architecture. Those can be found in the [Clean Architecture book](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164) and the [many](https://gist.github.com/ygrenzinger/14812a56b9221c9feca0b3621518635b) [articles](https://blog.ndepend.com/introduction-clean-architecture/) that have [already](https://medium.freecodecamp.org/a-quick-introduction-to-clean-architecture-990c014448d2) [been written](http://lmgtfy.com/?q=clean+architecture+by+uncle+bob).

### The problem details

Before I dive into the code, I will describe the problem I'm trying to solve. I want to automate making invoices, so that every month I have to do minimal work to get an invoice to send to my customers.

The data comes from my time keeping app [TimeCamp](https://www.timecamp.com/). They have a very nice service which includes an API so I can easily access my time data. From that data I can calculate how many days I've worked for which customer.

With that information, I can then generate an invoice in PDF format with all information about my company, my client and the invoice lines. This way I don't need to spend half an hour to an hour each month to fill out an Excel form to get an invoice. This might sound like a trivial time increase, but the experience from building this is equally valuable.

I encourage everybody reading this to experiment in their own time with a personal project and use techniques or technology you're not familiar with. They're awesome learning experiences and can be great poster projects when you want to advance your career.

Stay tuned because next week I will start detailing the architecture and the code of the application.
