---
title: "Injecting AutoMapper profiles in tests"
datePublished: Mon Jan 15 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iuxoo00xyyes1cd643mar
slug: injecting-automapper-profiles-in-tests
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


In the newer services at my client, [AutoMapper](http://automapper.org/) is used to map [DTO](https://en.wikipedia.org/wiki/Data_transfer_object)'s to database objects and back. Because mocking a mapping isn't obvious, a lot of behaviour wasn't tested and that's unacceptable. Let's find out how to properly inject an `IMapper` with actual mappings.

The full setup is that we create [`Profile`s](http://docs.automapper.org/en/stable/Configuration.html) that know how the mapping of objects happen. AutoMapper, combined with an [IOC](https://msdn.microsoft.com/en-us/library/ff921087.aspx) framework, automagically gathers all profiles and an `IMapper` instance is injected into the right places.

Now what happens when I want to test something that needs to be handled by the `IMapper` object. The _easiest_ way is to figure out where the object comes from, say a database or passed along to the method, retrieve a dummy object and let the mapper mock produce the real object that will be used in the test. This would look like this (C# pseudo code):

```
// test setup
var actualObject = new MyObject();
var parameters = new ParameterObject();
var mockRepository = new Mock<IRepository>();
new mockMapper = new Mock<IMapper>();
mockMapper.Setup(x => x.Map(It.Any())).Returns(actualObject);
new MyService(mockRepository, mockMapper).Execute(parameters);
 
class MyService
{
  void Execute(Any parameters)
  {
     var repoObject = _repository.Get();
     var actualObject = _mapper.Map(repoObject);
     var otherActualObject = _mapper.Map(parameters);
     // do stuff
  }
}
```

The biggest problem I have with this is that the `Profile` isn't tested. I've had multiple problems with bad mappings, missing mappings or where I had made a mistake in wiring it up. So I want to include the mapping in my tests without duplicating the mapping code.

After some googling, I found a surprisingly easy way to do this.

```
var myProfile = new MyProfile();
var configuration = new MapperConfiguration(cfg => cfg.AddProfile(myProfile));
var mapper = new Mapper(configuration);
```

It only takes three steps. The first is to create an instance of the mapper you want to use. The second step is to create a `MappingConfiguration` and register my `Profile`. The third and last step is to create an instance of a `Mapper` and pass the `MapperConfiguration` to the `Mapper`. This `Mapper` instance can then be passed to my test and will map with my custom `Profile`.

Now I have more confidence that my tests properly verify more of my code.
