---
title: "Not every class needs to be injected"
datePublished: Mon Mar 19 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iytaf0108zfs1hb6h60us
slug: not-every-class-needs-to-be-injected
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381131253/TXBS2EFDQ.jpeg
tags: csharp, software-architecture, dependency-injection, dotnet, dotnetcore

---


A trend I see in code that gets written by friends and coworkers is that every class needs to be injected. They all look so surprised when I tell them that they don't have to do that with every class they need. Then they always follow up with: then why do we need a [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) (DI) framework?

The simple answer is to make your life more easy, but that isn't always the case. Lets start from the beginning. When I see a new service class being created, most of the time it looks something like this:

```
public class ExcessiveInjection
{
  public ExcessiveInjection(ICreateName createName, IDoSomeWork doSomeWork, IRepository repository)
  {
    // store injected objects in properties
  }
}
```

The implementation of most of the interfaces is only used once, in the class they are being injected into. In the end this causes more work for both the programmer (who has to create unnecessary interfaces) and the PC running the application (more objects to store in the DI framework). This whole process can be simplified.

For me, there are two good reasons to create and inject an interface:

1. I want to mock particular functionality in my tests
2. I want to provide multiple ways of doing something

The most important principle is to [keep software simple and stupid (KISS)](https://en.wikipedia.org/wiki/KISS_principle) and to use techniques that will solve your problem, not multiply them. Getting a DI framework to do your bidding isn't always the easiest thing to do.

Before I delve into the DI remark, let me explain my two points. The first reason to provide an interface is when I want to exclude some functionality in my tests. This always pertains to external systems such as the operating system (OS), storage solutions (think database, document db, reddis cache, etc) or a call that needs to go to an endpoint somewhere on the network or internet. I want to focus my tests on the behaviour of the code, not test that I have working external services available. They might not be available on a build server that runs the tests as part of a gated check-in.

The second reason is when there are multiple ways of doing something. The easiest way to explain this is with an example. The project I'm currently working on is used in different countries. Tax is calculated differently in each country. I could inject an `ITaxCalculator` with different implementations per country: `BelgianTaxCalculator`, `FrenchTaxCalculator`, `EnglishTaxCalculator`, etc. Now I'm in a situation where one code block needs to perform different behaviour depending on the logged in user. If I cannot configure this in my DI framework of choice, then I can use a `TaxCalculatorFactory` that creates the correct calculator.

Does this factory need to be injected? Do I want to mock the factory in a test: no. I want to mock the database access, but not necessarily the factory itself. Do I want to provide multiple ways of implementing this factory: no. So I do not inject this factory. Here is how I would write this:

```
public class OrderService
{
  private readonly TaxCalculatorFactory _taxFactory;
  public OrderService(IRepository repository)
  {
    _taxFactory = new TaxCalculatorFactory(repository);
  }
  public void Process(int order)
  {
    var taxCalculator = _taxFactory.Create();
    // calculate tax with correct taxCalculator
    // process rest of order
  }
}
```

If I use the `TaxCalculatorFactory` in only one or two classes, then I'd just instantiate it in those classes. If I need it in three or more classes, then it might be interesting to let the DI framework inject it.

Another problem I see is keeping all code in one class can lead to a huge class. In huge classes that do several things, I get lost. So I encourage splitting functionality off into separate classes to increase code readability. Just instantiate the class in the method it's being used or in the constructor if it needs to be used in several places throughout the same class.

A little tight coupling is not always a bad thing. If only one part of my system needs to process an order, then I think it's overkill to split tax calculation off into a separate subsystem that needs to be injected. Just create some additional classes to hold the code you want to execute and keep it readable. The subsystem here is to process an order, not processing an order that needs a tax calculation subsystem that might need other subsystems.

The DI framework setup will be a lot smaller, comprehensible and will be less likely to contain bugs. I've encountered situations where a lot was injected, the DI framework wasn't properly setup and this caused memory leaks which slowed the application down to the point where it would crash. The cause was that I did not understood enough about DI frameworks.

When I set a DI framework up, it's not just registering interfaces and implementations. A lot of thought has to be invested into figuring out how long a piece of code can be kept into memory. Is it a smart choice to make Entity Framework `DbContext`s singletons? Is it overkill to create a new instance in every injection point? What about nested situations: I need the repository in both the `OrderService` and the `TaxCalculatorFactory`. If I inject the factory in the service, will two instances of the repository class generate concurrency exceptions? If/When they do, then finding the source of those bugs can be quite tricky. Dependency injection can simplify a lot of situations, but overuse will lead to complex bugs.

The lesson here is to inject with moderation. Remember the two rules (mocking the functionality and providing alternative implementations) when injection is useful and think how a change in code will impact the behaviour of the application.
