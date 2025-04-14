---
title: "Know the behaviour of your component"
datePublished: Mon Apr 10 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ivfsn00y0yes1g7tieq21
slug: know-the-behaviour-of-your-component
tags: programming, programming-tips

---


Good software consists of many moving components that interact well together. This lets you build one part at a time and it allows you to wrap your head around that part of the application. It then becomes important that the behaviour of the component being built is completely understood.

In order to build the component, it is important that the problem the component will solve is understood. For example: if I'm building an price calculator for a webshop, I need to know the number of items on the order, the price of each item, how much discount I need to apply, if I need to apply different discounts to different items or to the total price, maybe both, how much taxes I need to add for each item, whether the discount is applied before or after the taxes, etc.

Even for a simple component, there are a lot of questions that need to be answered. Those questions are specific for each component, custom to each organisation and piece of software. They will need to be asked to understand the behaviour of the application.

Let me emphasise the word _behaviour_ in that last sentence. Components can have dependencies on other components, ones that have possible not been designed or coded yet. In the example earlier, the tax calculation can be another component. I do not need to know how it works or where it gets the information for it's calculation. All I need to know is that there needs to be a point where the tax is calculated, because it is part of the behaviour of the component.

When I don't have a part for the component, like the tax calculation, I put my own interface in place that I can change, extend and implement later. As an added bonus, this lets met write tests for the component, while not all components are available or desirable in tests (think about repositories and external calls).

Details can change over time when you gain more understanding of the problem or when you find a more efficient way to solve the problem. Such technical details can be updated easily, but behaviour should only be changed by the business needs. An item that used to be 6% tax and must go to 21% tax isn't a behaviour change, that's just data. An optimisation or refactoring of the code isn't a behaviour change. No behaviour will be changed, added or removed. An example of a behaviour change is when there was only an option to give discount on the total amount and now business wants to apply a discount per item. This means that the behaviour of the application needs to be updated.

A problem arises when the not all behaviour is understood. Then the software will not meet the standards the client expects. This causes delays and in bad scenarios can lead to loss of trust between the team and the client (internal and external). That doesn't mean you cannot start working on the component before you know everything. It means that you shouldn't be afraid to ask questions to gain more insight so the software will meet the demands of the client.

Make sure you understand the problem and the expected behaviour. When you gain more insight, don't be afraid to revisit the code to improve it.
