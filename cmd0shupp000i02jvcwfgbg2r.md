---
title: "A simple A-Frame example"
seoTitle: "A simple A-Frame example"
seoDescription: "In this blog post, I'm creating a simple endpoint using a minimal API and applying A-Frame architecture."
datePublished: Sat Jul 12 2025 22:00:42 GMT+0000 (Coordinated Universal Time)
cuid: cmd0shupp000i02jvcwfgbg2r
slug: a-simple-a-frame-example
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747320140439/7308aa01-94bd-49cc-84d7-391f68871841.png
tags: software-development, software-architecture, architecture, a-frame, software-engineering, dotnet, dotnetcore, minimalapi, minimal-api

---

Now the expected structure is clear, let's take a look at the code. I will start with a minimal API implementation to demonstrate that there is no need for a framework to implement this architecture. I do know of a framework that makes A-Frame effortless to work with, I’ll go into more detail in the next blog post how this generates the controller code.

Let’s first get back to the simple scenario in my dog walking application. The initial endpoint will create a dog in the database. I'll start with the infrastructure code as this will be the most recognisable.

```csharp
record CreateDog(string Name, DateOnly Birthday);
app.MapPost(
    "/dog",
    async ([FromBody] CreateDog dog, [FromServices] DogWalkingContext db) =>
    {
        var existingDog = await db.Dogs.FirstOrDefaultAsync(d => d.Name == dog.Name && d.Birthday == dog.Birthday);

        var dogCreation = Dog.CreateDog(dog, existingDog);

        switch (dogCreation)
        {
            case DogCreated created:
                db.Dogs.Add(created.Dog);
                await db.SaveChangesAsync();
                return Results.Created($"/dog/{created.Dog.Id}", created.Dog);
            case DogExists exists:
                return Results.Redirect($"/dog/{exists.Id}");
        }

        return Results.InternalServerError("Could not determine what to do with the dog");
    })
.WithName("CreateDog");
```

The endpoint retrieves a possibly existing dog and passes it, together with the create command, to the `CreateDog` handle function. The logic function decides how to handle the creation. Either I create the dog in the database and sent a *201 Created* response or I send a *301 Redirect* response when the dog already exists. Should the logic returns something else, I return a *500 Internal Server Error* response.

Now that I have the infrastructure in place, let's look at the logic to create a dog in our system.

```csharp
abstract record DogCreation;
record DogCreated(Dog Dog) : DogCreation;
record DogExists(int Id) : DogCreation;

public class Dog
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateOnly Birthday { get; set; }

    internal static DogCreation CreateDog(CreateDog dog, Dog? existing)
    {
        if (existing is not null)
            return new DogExists(existing.Id);

        return new DogCreated(
            new Dog
            {
                Name = dog.Name,
                Birthday = dog.Birthday
            });
    }
}
```

If a dog already exists, I return the identifier of that dog. Otherwise, I'll create a new dog. When I remove infrastructure concerns, the remaining logic is straightforward. The return structure is easy to understand and describes all possible outcomes. I like this little pattern, it reminds me of [discriminated unions](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions) aka [union types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types). I hope [C# gets them soon](https://github.com/dotnet/csharplang/blob/main/proposals/TypeUnions.md) as I think functional programming paradigms are quite elegant.

Setting up automated tests for logic components is quite easy as there is no infrastructure that gets in the way. Infrastructure benefits more from integration tests that check all the messy side effects, while a (much faster) unit test can verify the output.

The readers who’ve been paying attention will have noticed that the controller and infrastructure code have interwoven in this example. Congratulations to those who spotted it. In this case, I don't mind as this is simple enough as a first example.

In more complex software I would put every infrastructure instruction into its own method or even class, except for Entity Framework queries. Entity Framework is an abstraction of the database, which hides the infrastructure details. I might put more complex queries in their own descriptive function, but I would not use the repository pattern.

Other infrastructure functionality would get their own class. This means that if I want to send an email, I’d create an `SendHelloEmail` class that would take care of sending out welcome emails. Depending on the needs of the software, I’d determine how abstract, flexible and configurable the setup of these infrastructure components should be.

## An even simpler scenario

The processing was easy enough to understand. I returned a reference to an endpoint that loads the details of a dog. How does that look in our A-Frame architecture? I wouldn't use A-Frame Architecture for this. Most queries are so straightforward that I don't want to bother with abstractions or indirection. I would just use a very, and I mean very, simple approach.

```csharp
app.MapGet(
        "/dog/{dogId}",
        async (int dogId, [FromServices] DogWalkingContext db) => Results.Json(await db.Dogs.FindAsync(dogId)))
    .WithName("GetDog");
```

Even if queries get more complicated, most don't reach the level of processing code. I'd place these in a separate file with a single function. There is the occasional exception that breaks this rule, but for those cases I'd look for a bespoke approach to the problem at hand instead of a one-size-fits-all approach. A one-size-fits-all approach either has the problem that it’s overly complex for simple scenarios or not flexible enough for the complex cases.

Now that the basics are clear, let's take a look at how I can make my life a lot easier with a framework that already does a lot of the heavy lifting.