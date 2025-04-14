---
title: "Do I know or did I just take a decision"
datePublished: Tue Dec 13 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0it0s000xayes1bt8y3tc0
slug: do-i-know-or-did-i-just-take-a-decision-2
tags: general

---


A while ago, the team asked who took the decision to use a certain pattern on how a custom configuration was implemented. As that happened weeks or even months ago, nobody could remember taking that decision. That got me thinking about when I take a decision versus when I know something for certain.

I asked myself how many decisions I take and whether it is my place to take them. There is a big difference between knowing something and deciding something. It's sometimes difficult to distinguish between the two.

The decision I referred to in the first paragraph was about adding custom configuration sections in a common project or adding a custom config section in the projects that need it. With the first solution, the projects that need custom configuration would depend on the common project that has one configuration class for each project. The other solution would mean adding a reference to `System.Configuration` in the relevant projects, which adds a full assembly for one feature.

The programmer that took that decision steered a part of the project in a direction without even thinking about it. Reversing this decision would take a bit of work for which we did not have time. Mixing the two is also not an option, before long nobody would know which is the correct way and it would lead to a lot of confusion, discussions and more time lost.

The programmer that took that decision might not have known he took one. It could be that it seemed logical, that that's the way it was done in previous projects or that made the most sense at the time. He thought he knew it was the correct thing to do.

This is not such an big change (albeit an inconvenient one), but think about a decision that would steer the project in a totally different direction: to use CQRS or how a business requirement should behave in a certain scenario. Now I hear some screams from the back that such things should be decided by the team lead or business people. When the project started and I was the first developer on the project, the team lead let me decide whether to use Entity Framework with either Database First or Code First approach. My decision took the project in a direction with benefits and pitfalls of its own. In that situation, the decision was explicitly left up to me. How many times since then have I taken decisions, no matter how small and insignificant they might have seemed, instead of knowing what to do or consulting with the rest of the team.

The difference in all these scenarios is that when you actually know something, then that steers the project into the general direction it is supposed to go. If you take a decision, you make the call in which direction the project should head. For technical issues, these decisions should be taken by the entire team where the team lead will have the final word if a consensus cannot be reached. Business requirement decisions should be taken by somebody from the business side of the project such as a functional analyst or an end user.

The point I'm trying to make is that if I'm not sure, if there is any doubt that I don't know for certain, I should take it up with the appropriate people so we can work it out together. This way, there won't be an argument in a few weeks where the team ultimately decides to reverse the decision and adds more work to the backlog.
