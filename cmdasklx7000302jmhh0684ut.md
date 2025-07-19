---
title: "Leverage Wolverine's A-Frame architecture support"
seoTitle: "Leverage Wolverine's A-Frame architecture support"
seoDescription: "Find out how the Wolverine framework can help me write simpler code using A-Frame architecture."
datePublished: Sat Jul 19 2025 22:00:33 GMT+0000 (Coordinated Universal Time)
cuid: cmdasklx7000302jmhh0684ut
slug: leverage-wolverines-a-frame-architecture-support
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747320233624/2c515955-a1e9-4ef6-a4c6-0c72768f04a2.png
tags: software-development, software-architecture, architecture, a-frame, software-engineering, dotnet, dotnetcore, Wolverine

---

I’ve shown how I write code using the A-Frame architecture without help. Yet I find a library or framework very convenient when using advanced techniques. [Wolverine](https://wolverine.netlify.app) is at its core a messaging framework, but goes well beyond that. It has an in-memory transport, so it behaves like a mediator, but with all the quality of life improvements that apply to any other transport. It supports durable messaging via the in- and outbox patterns, retries, timeouts, error handling and much more. The in-memory transport is great for integration testing as well. Once the app goes live, it can switch the transport out for an external system such as [RabbitMQ](https://www.rabbitmq.com), [Azure Service Bus](https://azure.microsoft.com/en-us/products/service-bus/), [Amazon SQS](https://aws.amazon.com/sqs/)/[SNS](https://aws.amazon.com/sns/), [Apache Kafka](https://kafka.apache.org) and even [SQL server](https://www.microsoft.com/en-us/sql-server) or [PostgreSQL](https://www.postgresql.org) if I have limited messaging needs. It also supports messages coming in through HTTP requests, so I can build an API with it.

Wolverine installs alongside other messaging and API frameworks such as [MassTransit](https://masstransit.io), [NServiceBus](https://particular.net/nservicebus), [minimal API](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/overview?view=aspnetcore-9.0), [ASP.NET](https://learn.microsoft.com/en-us/aspnet/overview) and [Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor). This allows gradual integration within existing projects. For this purpose, I will leave the functionality of creating and retrieving a dog in the minimal API.

Let me demonstrate how simple Wolverine can be with another simple example: a walk. To create a walk, I have to map the path I walked and which dogs I had with me. In a real system, I'd look for GPS integration. For this demo system I use a list of coordinates `[(0,0), (0,1), (2, 1), ...]`. I also want to verify that the dogs are in the system, so I don't have unknown dogs with me.

```csharp
public class RegisterWalkHandler
{
    public Task<Dog[]> LoadAsync(RegisterWalk request, DogWalkingContext db) =>
        db.Dogs.Where(d => request.DogsOnWalk.Contains(d.Name)).ToArrayAsync();

    public ProblemDetails Validate(RegisterWalk request, Dog[] knownDogs)
    {
        var knownNames = knownDogs.Select(d => d.Name);
        var unknownDogs = request.DogsOnWalk.Except(knownNames).ToArray();
        return unknownDogs.Any()
            ? new ProblemDetails
            {
                Status = (int)HttpStatusCode.BadRequest,
                Title = "Unknown dog or dogs",
                Detail = string.Join(", ", unknownDogs)
            }
            : WolverineContinue.NoProblems;
    }

    [WolverinePost("/walk", Name = "Register Walk", OperationId = "Walk")]
    [Tags("MoreThanCode.AFrameExample")]
    public (LazyCreationResponse<WalkResponse> response, EntityFrameworkInsert<WalkWithDogs> insertWalk) Handle(
        RegisterWalk request,
        Dog[] dogsOnWalk)
    {
        var walk = new WalkWithDogs
        {
            Dogs = dogsOnWalk,
            Path = request.Path.Select((coord, index) => new CoordinateEntity
                {
                    X = coord.X,
                    Y = coord.Y,
                    SequenceOrder = index
                })
                .ToList()
        };

        var response = () => new WalkResponse(walk.Id, dogsOnWalk.Select(d => new DogResponse(d)).ToArray(), request.Path);
        return (LazyCreationResponse.For(() => $"/walk/{walk.Id}", response),
            new EntityFrameworkInsert<WalkWithDogs>(walk));
    }
}
```

Wolverine works by convention: first it loads data when it sees a `Load` or `LoadAsync` function, then it validates the request, and finally it processes the `Handle` or `Consume` function. The `Load` function is equivalent to the infrastructure code, and the `Handle` function is my logic code. The `Validate` function is a bonus to even further separate concerns.

Through source generation, it creates the controller for me. This means that the code is fast because there is no reflection during runtime. It also figures out what to inject into each function. It can find injectable services through the configured dependency injection framework, it can inject messages from the bus and HTTP pipeline (including query parameters and form data) and objects returned from the `Load` function. That is how the `Validate` and `Handle` methods get the request and the list of dogs.

What I find most impressive is that Wolverine can handle the output of the `Handle` function. In an HTTP handler, the first item in the returned tuple is the response to the client. The next items will be either messages to be sent to the configured bus or handled as a side effect. A side effect is anything related to infrastructure that is part of the transaction: saving contents to a file, updating rows in the database,... It makes this distinction by looking for the presence of the `ISideEffect` marker interface.

For other side effects, especially concerning external systems, I put a command on the bus and have a handler interact with the external system. This has the benefit that it runs asynchronously, it’s possible to distribute processing and it uses built-in resiliency and retry mechanisms. It also promotes reuse as multiple components can trigger the integration with a simple message.

These are the most important features to get started with A-Frame architecture. For more in-depth knowledge I'll refer to the [Wolverine docs](https://wolverine.netlify.app/tutorials/).

Next up, I’ll dive into a more complex example.