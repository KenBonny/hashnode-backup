---
title: "What is A-Frame architecture?"
seoTitle: "A code agnostic overview of A-Frame architecture"
seoDescription: "A-frame architecture separates interfacing with infrastructure from taking decisions using logic. A controller orchestrates communication between the two."
datePublished: Sun Jul 06 2025 06:00:30 GMT+0000 (Coordinated Universal Time)
cuid: cmcr9jwnz000m02l8fj5q9852
slug: what-is-a-frame-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747320099684/4782f1d0-73b5-4c90-a270-df1b6e9aa4b8.png
tags: software-development, software-architecture, architecture, a-frame, software-engineering, programming-ciovqvfcb008mb253jrczo9ye

---

A-frame architecture is a pretty simple [architectural pattern](https://en.wikipedia.org/wiki/Architectural_pattern): it separates interacting with infrastructure from taking decisions using logic. Between the two is a controller who orchestrates the flow of data. Everything in your code should respect that separation. As this pattern talks more about how to structure code inside a component, it’s best used in combination with an architecture that describes how to structure components. I find that [Vertical Slice](https://www.jimmybogard.com/vertical-slice-architecture/), [Modular Monolith](https://www.youtube.com/watch?v=5OjqD-ow8GE) or [Microservices](https://martinfowler.com/microservices/) architectures pair well with A-Frame architecture.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747318005509/ac0772c9-9e95-4a44-ad75-37c21856b781.png align="center")

This leads to nicely separated code components, have a single responsibility and have no dependencies on other components. This makes each component easy to reason about, which in turn leads to code that is easily tested, changed and replaced. Infrastructure components tend to be more general and promote reuse, while logic is more specific to each use case.

For example, I can have several logic components that need to write an image to the file system. One will deal with profile pictures while another will handle uploaded photographs. Both will delegate the write operation to the same infrastructure component.

### Infrastructure

Infrastructure components (aka infrastructure) interact with external systems, read or write state and call functions with an unpredictable outcome. Examples are:

* Database calls
    
* Sending requests over the network
    
* Reading or writing to the file system
    
* Retrieving environment variables
    
* Determining date and time
    
* Generating random numbers, ids or [uuids](https://en.wikipedia.org/wiki/Universally_unique_identifier)
    

These components can be harder to test without an actual system to talk to. They’re challenging to mock or replace, both in automated tests and test environments. Infrastructure can also behave in unpredictable ways: do I have the right permissions or credentials, is there enough space on a disk, can I make the call through the firewall, etc.

That is why I like to wrap these systems in an abstraction that I can more easily control. Sometimes I create my own interfaces and implementations. For file system access I mostly always create an `IFileSystem` interface that have `Read` and `Write` methods, sometimes with (de)serialisation baked in. When there are good abstractions already available, I reuse those. [`IOptions`](https://learn.microsoft.com/en-us/dotnet/core/extensions/options) is invaluable for accessing settings and [`TimeProvider`](https://learn.microsoft.com/en-us/dotnet/standard/datetime/timeprovider-overview) is a great way to abstract time management.

When it comes to database access, there is the [repository pattern](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design#the-repository-pattern). This is a good option when I access the database directly with [ADO.NET](http://ADO.NET) or [Dapper](https://www.learndapper.com). I don’t recommend using the repository pattern together with [entity framework](https://learn.microsoft.com/en-us/aspnet/entity-framework) for the simple reason that a `DbContext` already is an abstraction over the database. I've seen this lead to a maze of indirection and duplication of logic. A repository with `Repository.Get(Expression<Func<Model>> where)` that has one implementation which just forwards the `where` to the `DbContext`, comes to mind.

When I suggest removing the unnecessary repository pattern, I receive these two arguments against it:

1. *What about reusing a query?* In my experience, most queries are unique. Extension methods are a great place to store reusable statements. Think filtering by a status enum or by a date range. What I'm trying to avoid is a single function with parameters for each case. `Search(StatusEnum? status, DateRange? bornBetween, int? idToFilterOn)` with `if`'s throughout the body to filter by each optional parameter. The implementation will get quite complex and confusing. Not to mention the dozens of tests to check that it works with every combination. When there is a search endpoint that needs to perform this kind of complex query, I make a specific endpoint with the complex logic inside. The code for these queries is generally not reused.
    
2. *How do I test against the* `DbContext`*?* Instead of using mocks/fakes/stubs, use a real database. This is what integration tests are for as there is no substitute for a real database. See the [Testing A-Frame architecture](https://kenbonny.net/testing-a-frame-architecture-with-tunit) article for a detailed explanation.
    

Keep infrastructure as simple and straightforward as possible. I prefer to have them as standalone components that do one thing and do it well. For example, a `FileSystem.Write(Image image)` should know how to serialise the image to a byte array. If there are multiple ways of serialising, then either the component can determine what serializer to use or the logic code takes that decision and passes the serializer or the serialised content to the file writer.

In infrastructure code, observability is my best friend. No matter how much I prepare and test, the real system will throw curveballs my way. That is why all infrastructure components should instrument [OpenTelemetry](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel) so I can track requests throughout systems.

### Logic

Now that I've loaded data, it's time to make decisions based on that information. This is where logic components come into play. An alternative term is business logic, but I prefer the more generic term to keep it applicable to more scenarios.

A logic component is, preferably, a [pure function](https://en.wikipedia.org/wiki/Pure_function). It takes the data it needs as input and returns the decisions it has made. Most logic components are going to be quite easy to read and understand. The most important rule of logic components is that they can’t access external systems. There are a lot of similarities between logic components and the domain model from [domain driven design](https://martinfowler.com/bliki/DomainDrivenDesign.html) practices.

Infrastructure components handle the decisions that logic components take: save data to a database, write data to a file system, notify external systems and post messages to a bus. The only exception I make is for logging as this is tightly coupled to the logic flow. I could return the log events and write them to a log stream in an infrastructure component. I find that going this far is overkill and complicates the flow. An alternative to logging is writing the in- and output of logic components to OpenTelemetry. This keeps the logic free of logging statements and still gives me all the information necessary to debug later.

When I have need of external libraries in my logic code, I look for ones that don’t produce side effects. For example, an image processing library should take the image as input and return it in the same format as output, I don’t want it saving the image to the file system. If the library is complex, I hide that complexity in its own class. Say I need to add a watermark to an image. I'll wrap the extensive image processing library in a class called `Watermark`. Injection or instantiation then depends on how expensive it is to create that class. I instantiate a read-only field `private readonly Watermark lib = new();`, I create it inside the function `var lib = new Watermark();` or I inject them after the data `Process(Model model, DateTime now, Watermark lib)`. I don't mind tight coupling if it makes sense. When testing this functionality, I automatically test that the logic component calls the library correctly. Because I don’t allow infrastructure, they’re still easy to set up in my tests.

This approach lends itself to reuse very easily: inject or instantiate the `Watermark` class and use it. It's even easy to extend to add a timestamp in another feature... Wait, hold that thought. I get why this seems like a good idea, both are adding something to an image. Unfortunately, this is a case where the functionality looks alike but is quite different in practice. Adding a watermark is something else than adding a timestamp. They’re only accidentally alike. The moment I'd start implementing this, I'd notice they’re quite different. I would take the lessons learned from the `Watermark` implementation and just create another class `Timestamp`. This is easier to maintain, evolve, replace or compose.

It's only when I notice that similar code appears in the codebase that I'll reflect and refactor into a shared class or component. The difference is that I'll react to what is actually there instead of prematurely optimising. This way it's more challenging to create accidental complexity.

### Controller

This is a good point in the development process to think about the last step. I can load the necessary data and act upon it; all that I need to do is to connect the dots. This is where the controller comes into play. It will pass information from one to the other and make sure the two never meet. The controller will determine what data to load and pass it on to the logic component. Finally, it will instruct other infrastructure components based on the output of the logic component. This code is fairly straightforward and can even be automated away.

In practice, controllers are endpoint declarations, message handlers, WPF binding methods, cronjob entry points or equivalent. This is the place that will know where to get the data from and which logic component to pass it to.

Let’s put this theory into practice with a simple example.