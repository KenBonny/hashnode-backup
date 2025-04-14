---
title: "Ways to rejuvenate old code"
datePublished: Mon Feb 13 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j4i7y011myes1hmvldsvd
slug: ways-to-rejuvenate-old-code
tags: refactoring, programming, general

---


Some systems have accumulated so many hacks, workarounds and cruft that improving the codebase has more in common with a Herculean task than anything else. I see two possible ways to go about this task.

When a system is modified for years on end with no clear direction, architecture, source control and developers who don't keep up with new techniques, a big ball of mud is created. One where quick and dirty solutions are chosen above reusable, reliable code. It's exacerbated when the same software is copied and modified for different customers instead of applying a multi-tenant or extendable solution.

To get the software in shape again, I see two ways forward. Both will take a great amount of effort, time and resources. In both scenarios, providing training for developers is important because everybody has to be on the same page to change the software for the better. No more shortcuts, hacks or workarounds should sneak into the improvements.

The first option is to refactor the code line by line, making slow and deliberate improvements. These improvements should be accompanied by copious amounts of unit tests to make sure everything works as intended. The biggest advantage of this approach is that all current functionality is kept and improved over time. The biggest risk is that the software is still being sold to more customers and the improvements take a backseat to satisfying the customer. This will lead to complications and delays caused by requests for change or new features demanded by said customer. Those changes take more time than they should depending on the progress already made on refactoring of the old code.

The second option is to start over, leaving the old product behind and supporting it for existing customers. The new product would then use the domain knowledge gathered from the old software, but will be implemented with a better architecture and new programming techniques. The upfront cost of this approach is higher than refactoring the old codebase because the new software cannot be sold until the base functionality is re-implemented. Starting over means that the rejuvenation cannot be put on hold or sidestepped because of new clients.

Besides the cost, there is another risk that should not be underestimated when choosing the second option. The new software should not be regarded as a replacement of the old software, but as a new product that will evolve on its own. The features available in the old software got there over the course of years. Unfortunately, with each change, the old software grew a bit more stale and difficult to customise. The features of the old software should serve as basis for the new software. This time, care should be put into making sure clean code is produced, no shortcuts are taken and cruft does not accumulate.

A good example is the .net framework. The original framework is now up to version [4.6.2](https://www.microsoft.com/net) (at time of writing), but they also started over with [.net core](https://www.microsoft.com/net/core). Let me clarify that I don't think the old framework is bad, but it was time for a makeover. Microsoft is adding new features to the "old" framework to keep it relevant, but they weren't afraid to start over and build functionality from the ground up. They did it with an attitude of "this is not regular .net, but a new, independent version". In .net core version 1, there were features missing and tooling wasn't up to date. They kept updating .net core and changing it independently from the old framework. In my opinion, it's [paying off big time](https://www.ageofascent.com/2016/02/18/asp-net-core-exeeds-1-15-million-requests-12-6-gbps/).

I want to point out that Microsoft has the resources to maintain 2 products of this size. A lot of companies won't have the budget to maintain 2 products and will have to choose the way forward for them. Whichever option they choose should be encouraged to run to completion. Stopping halfway will only increase the ball of mud, because it's one more half-finished attempt to improve an ageing codebase.

Microsoft took its time to get it right and they weren't afraid to make changes (and boy, were there changes). The result so far is very impressive. A very loosely coupled system that is more optimised and extendable than ever. A lot of companies can learn from this example.
