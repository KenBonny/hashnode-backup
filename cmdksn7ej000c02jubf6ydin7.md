---
title: "Tackling complex examples using A-Frame architecture and Wolverine"
seoTitle: "Tackling complex examples using A-Frame architecture and Wolverine"
seoDescription: "The examples in the previous posts seem really nice for simple scenarios. How do I approach more advanced use cases?"
datePublished: Sat Jul 26 2025 22:00:15 GMT+0000 (Coordinated Universal Time)
cuid: cmdksn7ej000c02jubf6ydin7
slug: tackling-complex-examples-using-a-frame-architecture-and-wolverine
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747320177682/5139137e-d252-4802-ba1d-1fea0ab4f687.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1747319633038/58470daa-a20a-484e-8c9f-d0939482f15e.png
tags: software-development, software-architecture, architecture, a-frame, software-engineering, dotnet, dotnetcore, Wolverine

---

The examples in the previous posts seem really nice for simple scenarios. How do I approach more advanced use cases? The big scenarios are:

1. I need multiple pieces of data from different sources
    
2. I need to perform an infrastructure call in the middle of a logic component
    
3. I need to do different infrastructure calls based on the decisions taken by my logic code
    

### 1\. Multiple pieces of infrastructure data

When multiple objects need to be loaded, the `Load` method can return a tuple. Wolverine injects each object separately in the `Validate` and `Handle` methods. The `Load` function returns the dogs from the database, an image from the web and the current date from the system clock as a `Tuple`. Wolverine will inject the list of dogs, the picture and the date correctly into the `Validate` and `Handle` methods. If different functions require different objects, they only need to specify which data they require. Wolverine will even resolve the services such as the `IWatermarkService` from the dependency injection framework.

```csharp
public async Task<(List<Dog> dogs, Image picture, DateTimeOffset now)> LoadAsync(/* dependencies go here */) 
{
    // infrastructure code goes here
    return (dogsFromDatabase, pictureFromTheWeb, DateTimeOffset.Now);
}

public ProblemDetails Validate(List<Dog> dogs, DateTimeOffset now)
{
    // validation code goes here
    return WolverineContinue.NoProblems;
}

public DogDto Handle(List<Dog> dogs, Image picture, DateTimeOffset now, IWatermarkService watermark)
{
    // logic code goes here
}
```

### 2\. Infrastructure calls in the middle of logic code

The easiest solution is to avoid this scenario. Try to structure the logic code differently so that the infrastructure code can load all data that upfront.

There are situations where I don't want to incur the upfront cost. For example, when there is a chance that the data isn't necessary and the load puts the system under stress. In such cases, I can inject infrastructure code into the logic component. This does not need to mean that I'm back to square one of injecting an interface or `DbContext` into my `Handle` method.

`Func<>` can be returned from the `Load` method. Thus delegating the execution to a later time. I get the benefits that the infrastructure code prepares the call, and I get an immutable way of testing my logic as I can replace the `Func<>` with a simple test stub.

```csharp
public async
    Task<(WalkWithDogs? Walk, List<WalkWithDogs> OtherWalks, Func<byte[]> GetPictureAsync, DateTimeOffset Now)>
    LoadAsync(int walkId, DogWalkingContext db)
{
    var walk = await db.WalksWithDogs.Include(w => w.Dogs).FirstOrDefaultAsync(w => w.Id == walkId);
    var dogsInWalk = walk?.Dogs.Select(d => d.Id).ToArray() ?? [];
    var otherWalks = await db.WalksWithDogs.Include(w => w.Dogs)
        .Where(w => !w.Dogs.Any(d => dogsInWalk.Contains(d.Id)))
        .ToListAsync();
    var getPicture = () =>
    {
        var stream = Assembly.GetExecutingAssembly().GetManifestResourceStream("MoreThanCode.AFrameExample.Yuna.jpg");
        if (stream == null)
            return [];
        stream.Seek(0, SeekOrigin.Begin);
        byte[] image = new byte[stream.Length];
        stream.ReadExactly(image);
        return image;
    };
    return (walk, otherWalks, getPicture, DateTimeOffset.Now);
}

[WolverineGet("/friends/{walkId}", OperationId = "Friends-On-Walk")]
[Tags("MoreThanCode.AFrameExample")]
public IResult Handle(
    WalkWithDogs walk,
    List<WalkWithDogs> otherWalksAtSameTime,
    Func<byte[]> getPicture,
    DateTimeOffset now,
    Watermark watermarkService)
{
    if (!otherWalksAtSameTime.Any())
        return Results.Empty;

    var friends = otherWalksAtSameTime.SelectMany(w => w.Dogs)
                    .Except(walk.Dogs)
                    .Select(d => d.Name)
                    .ToArray();

    FriendsResponse response = friends.Length == 0 
        ? new([], []) 
        : new(friends, watermarkService.Add(getPicture()));

    return Results.Ok(response);
}
```

### 3\. Complex return instructions

What if I want to save the walk back to the database, write the picture to disk and publish additional messages? For these purposes, Wolverine has [cascading messages](https://wolverine.netlify.app/guide/handlers/cascading.html) and [`ISideEffect`s](https://wolverine.netlify.app/guide/handlers/side-effects.html).

Although there are other ways to return cascading messages, I prefer to work with the `OutgoingMessages` response type. This way I can return none, one or multiple messages based on the decisions of my logic. Wolverine will publish these messages to the bus according to the configured routing information. It's also possible to schedule or delay messages if needed. All these messages will benefit from built-in resiliency mechanisms.

Infrastructure operations that are part of the scope of the handler are best implemented as side effects. Common side effects are saving to the database and writing to disk. For this, there is the `ISideEffect` marker interface that expects an `Execute` function to be available. This function is not present on the interface as the input can accept parameters resolved from dependency injection, just like the `Handle` function discussed earlier. When a side effect is optional, make it nullable and return `null` if it should not happen.

It isn't possible to return a generic list of `ISideEffect`s. Wolverine needs to know the explicit type of the side effect to resolve the injectable parameters correctly. Side effects are part of the transaction spanning the `Load`, `Verify` and `Handle` functions. This means that if a side effect fails, the message processing fails.

Wolverine publishes the messages after everything in the transaction succeeds. This prevents ghost messages notifying other parts of the system of an operation that may have failed. To prevent messages from disappearing because there is a problem with the bus, I recommend enabling the outbox via [durable messaging](https://wolverine.netlify.app/guide/durability/). Storing the message in the database is part of the transaction which prevents those messages from being lost.

```csharp
public (IResult, OutgoingMessages, EntityFrameworkInsert<WalkWithDogs>?) Handle(
    WalkWithDogs walk,
    List<WalkWithDogs> otherWalksAtSameTime,
    Func<byte[]> getPicture,
    DateTimeOffset now,
    Watermark watermarkService)
{
    var outgoingMessages = new OutgoingMessages();

    if (!otherWalksAtSameTime.Any())
        return (Results.Empty, outgoingMessages, null);

    var friends = otherWalksAtSameTime.SelectMany(w => w.Dogs).Except(walk.Dogs).Select(d => d.Name).ToArray();
    if (friends.Length != 0)
        outgoingMessages.Add(new MetFriends(friends));

    FriendsResponse response = friends.Length == 0 
        ? new([], []) 
        : new(friends, watermarkService.Add(getPicture()));

    return (Results.Ok(response), outgoingMessages, new EntityFrameworkInsert<WalkWithDogs>(walk));
}
```

A last remark: when a side effect can fail, publish an event or command and handle it. The most prevalent examples are network calls. A network call can fail for a multitude of reasons. When I trigger the call to an external system in an event, I can leverage built-in [retry](https://wolverine.netlify.app/guide/handlers/error-handling.html) and [error](https://wolverine.netlify.app/guide/handlers/error-handling.html) handling mechanisms. This means I have battle-tested ways of handling failures.

Next up, I’ll be looking into different ways of testing logic and infrastructure code.