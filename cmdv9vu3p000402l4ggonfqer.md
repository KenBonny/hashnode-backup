---
title: "Testing A-Frame architecture"
seoTitle: "Testing A-Frame architecture"
seoDescription: "No architecture is complete without an easy way to test the functionality. My recommended strategy is to use two types of tests: unit and integration."
datePublished: Sun Aug 03 2025 06:00:33 GMT+0000 (Coordinated Universal Time)
cuid: cmdv9vu3p000402l4ggonfqer
slug: testing-a-frame-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747320200528/e83794dd-8a67-48ce-82f4-1a32d6cdb84d.png
tags: software-development, software-architecture, architecture, testing, a-frame, software-engineering, software-testing, dotnet, dotnetcore, t-unit, tunit

---

No architecture is complete without an easy way to test the functionality. My recommended strategy is to use two types of tests: unit and integration.

As far as test setup goes, there is no clear winner for a test framework. [xUnit.NET](http://xUnit.NET), [NUnit](https://nunit.org) and the newcomer [TUnit](https://tunit.dev) are all dependable frameworks. Personally, I like xUnit for their terminology with Fact and Theory, yet the newcomer TUnit is quickly capturing my interest with some [interesting features](https://tunit.dev/docs/comparison/attributes#test-control-attributes). I've used it here to encourage everybody (including me) to keep experimenting and learning.

I have similar thoughts on assertion and mocking libraries. There are no bad choices. They’re just tools to get the job done. Pick one and use it as long as it is useful.

### Unit tests

The logic code is the easiest to test with standard unit tests. These tests will verify all scenarios that the software needs to support. Since there is no infrastructure setup required, these tests are easy to read and fairly straightforward. Since unit tests are pretty cheap (quick) to run, I’ll write a lot of them to cover all scenarios.

Instantiating the system under test is nothing more than creating a new instance of the class. Here I see the first A-Frame benefit: by limiting the number of services to inject, I can simplify my test setup. When there are no systems to prepare and ensure they return the correct data in the correct circumstances, it takes a gigantic load off my shoulders when preparing my tests.

All injections happen in the `Handle` function. Keep injected data simple and specific to the case under test. In this case, I create a default `_walk` that I pass to the function or that can serve as an [object mother](https://martinfowler.com/bliki/ObjectMother.html) that I modify to the needs of the test. The `_otherWalk` is similar to the `_walk` object, but with another dog that crossed paths with ours. The `Func<byte[]>` is an easily stubbed method that captures whether the function has been called. Should the need arise, returning different data is now trivial.

While the date is irrelevant for this code, it is a good example to keep mutable calls out of the logic component. Testing with specific dates has been a challenge in the past as there could be a hardcoded [`DateTimeOffset.Now`](http://DateTimeOffset.Now) present. It is possible to replace that with the [`TimeProvider`](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider). I prefer simply passing the date to the test instead of setting up another system.

The result of the logic tells me what it expects to happen. I don't need to start checking multiple mocks and stubs, simple asserts are enough. While there is a mock here, it is only one and not multiple. I find that the result is transparent to interpret.

```csharp
[Test]
public async Task When_other_dog_encountered_then_do_indicate_dog_encountered()
{
    var getPictureCalled = false;
    var (result, outgoingMessages, entityFrameworkInsert) = new MetFriendsHandler().Handle(
        _walk,
        [_otherWalk],
        () =>
        {
            getPictureCalled = true;
            return [];
        },
        DateTimeOffset.Now);

    await Assert.That(result).IsEquivalentTo(Results.Ok(new FriendsResponse(["Toby"], [])));
    var friends = outgoingMessages.ShouldHaveMessageOfType<MetFriends>().Friends;
    await Assert.That(friends).IsNotEmpty().And.Contains("Toby");
    await Assert.That(entityFrameworkInsert).IsNotNull();
    await Assert.That(getPictureCalled).IsTrue();
}
```

### Integration tests

Now that I've tackled the easy part, let's look at the complex part. Infrastructure code is not easy to test, no matter how I twist or turn it. I've tried mocking it out, I've tried using the Entity Framework in-memory database, I've tried sacrificing managers to the god [Maniae](https://en.wikipedia.org/wiki/Maniae). This is where integration tests shine. These tests verify how your system responds to actual external system behaviour. They're slower than unit tests, so focus on the most critical scenarios while using unit tests for edge cases.

Integration tests run (as close to) actual systems. This means starting your application in a web server, talking to a real database and sending requests to external systems over the network. To run the web server in-memory, I use the [WebApplicationFactory](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests). This is what I did in my basic (and honestly, quite useless) integration test. In more complex scenarios, I use a library such as [Alba](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests) or [Playwright](https://playwright.dev/dotnet/).

[Don't try to simulate the database](https://learn.microsoft.com/en-us/ef/core/providers/in-memory/?tabs=dotnet-core-cli). I create a local test database in a container or spin up a lightweight database in my CI/CD pipeline. [Test containers](https://testcontainers.com) can be quite convenient, but they take some time to start. I run my migration scripts, then I test against that database. This way, I ensure that my migration scripts work on the database. Another system that works as expected. To reset a database to a known good point, I use [Respawn](https://github.com/jbogard/Respawn). I prefer resetting the database before each test run. This ensures a clean database before each test, which populates it with just the necessary data and eliminates a previous test from influencing another. After a test fails, I have access to the data to debug efficiently.

Write to the local file system when integration testing. In a CI/CD environment, my tests run in a container that gets disposed of afterwards. This is a good environment to try writes as the data gets discarded after every run. I can even publish the test output as an artefact if I want to inspect it afterwards.

The only time I mock, fake or stub interfaces is when I work with external services. To get realistic responses, I call each system with test data. I prefer doing this with a test system, but I will use the real API if I have no other option. In the last case, I'll never do that unannounced. I'll get in touch with the external services team to coordinate a moment and specify which data I’ll send. In all cases I keep the requests and the responses. I use both good and bad responses in my integration tests, so my system is prepared for all possible scenarios.

Practically, I try to mock, fake or stub the `HttpClient` to get control of its return values. [WireMock.NET](http://WireMock.NET) can come in quite handy in these scenarios. If I use libraries such as [Refit](https://reactiveui.github.io/refit/), [RestSharp](https://restsharp.dev) or [Flurl](https://flurl.dev), I'll use their built-in test support.

```csharp
[ClassDataSource<WebAppFactory>(Shared = SharedType.PerTestSession)]
public class MetFriendsIntegrationTests(WebAppFactory webAppFactory)
{
    [Test]
    public async Task Get_response_bad_request()
    {
        var client = webAppFactory.CreateClient();

        using var response = await client.GetAsync("/friends/1");

        await Assert.That(response.StatusCode).IsEqualTo(HttpStatusCode.NotFound);
    }
}

public class WebAppFactory : WebApplicationFactory<Program>, IAsyncInitializer
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // let Oakton accept --environment variables
        OaktonEnvironment.AutoStartHost = true;

        // disable all external setup so the integration tests don't start sending out messages
        builder.ConfigureTestServices(services => services.DisableAllExternalWolverineTransports());
    }

    public Task InitializeAsync()
    {
        // Grab a reference to the server
        // This forces it to initialise.
        // By doing it within this method, it's thread safe.
        // And avoids multiple initialisations from different tests if parallelisation is switched on
        _ = Server;
        return Task.CompletedTask;
    }
}
```

Next up, the last post in this series: my closing thoughts on A-Frame architecture.