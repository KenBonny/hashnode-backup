---
title: "Unit testing part 7: Wiring it all up"
datePublished: Tue Oct 11 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j308i011nzfs12h830txw
slug: unit-testing-part-7
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618404803973/uUSIx0lgd.png
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


_This [series of posts](https://kenbonny.net/tag/better-code-with-tests/) accompanies a talk I give about unit testing with the [xUnit test framework](https://xunit.github.io/). Any reader who saw my talk can use this as a reference, others can use this as a starting point to write even better maintainable and reliable code. In my talk, I highlight patterns that work well with easily testable code and combine it all into a working application. All code from these articles can be found in a [repository](https://github.com/KenBonny/IronMan) on my GitHub account._

I now have some very beautiful code. Well written, fully tested, every part does what it's supposed to do. Now what?

## Wiring it all up

I don't want to write a dozen `new Launcher(new Collection&lt;IMissile&gt; {new LaserGuidedMissile()}` instances, never mind what will happen if I need to change or add an interface somewhere.

Enter [dependency injection](https://msdn.microsoft.com/en-us/library/dn178469(v=pandp.30).aspx) (DI): I define what interface will resolve to what class and how long the object can live. Then I just ask the framework to give me an instance of a certain interface to work with. For example: in a website, I could ask the DI framework for a new connection to the database with each new request. If two requests happen at the same time, the framework will instantiate two instances that can work independently. All the power resides with how I want to set up my environment.

There are a lot of frameworks I can use in .net: [Castle Windsor](http://www.castleproject.org/), [Unity](https://github.com/unitycontainer/unity), [Autofac](https://github.com/autofac/Autofac) and [many more](https://www.google.be/search?q=.net+dependency+injection+frameworks&tbs=qdr:y)... Each [more performant](http://www.palmmedia.de/Blog/2011/8/30/ioc-container-benchmark-performance-comparison) than the next. For this project I will use [LightInject](http://www.lightinject.net/) since I have not used this before and am curious how easy it is to use. I have used Autofac and Unity in work projects and they work like a charm.

After some reading, I configured the container in the new `IronMan.Jarvis` console project as follows:

```
internal static IServiceContainer ConfigureContainer()
{
  var container = new ServiceContainer();
  // normal registration
  container.Register<ILauncher, Launcher>();
  // named registration
  container.Register<IMissile, HeatSeekingMissile>(MissileType.Autonomous.ToString());
  container.Register<IMissile, LaserGuidedMissile>(MissileType.Guided.ToString());
  // mass registration
  container.RegisterAssembly(typeof(WeaponsAssembly).Assembly,
    (serviceType, implementingType) => serviceType.IsAssignableFrom(typeof(IMissileFactory)));
  return container;
}
```

I added the `ILauncher` and `IMissileFactory` interfaces to the corresponding classes to make this more easy. I also added the empty class `WeaponsAssembly`.

There are a couple of things going on in there that cater to different situations. Lets start with the simpler situations first. The first line registers the `Launcher` class as `ILauncher` interface. So if I ask for an `ILauncher` I will get an instance of the `Launcher` class. If I ever want to change the launcher, then I can just build a new class, add it here and it will be used instead of the previous one. All of the new dependencies will be resolved by the DI framework.

The next two lines register the `HeatSeekingMissile` and `LaserGuidedMissile` as named missiles. This means that when I would resolve just `IMissile`, I would always need to specify a name. If I tried to resolve the interface `IMissile`, I would get an exception. Because I registered both missiles with their `MissileType` as name, I can easily ask for either, using that name. This might sound familiar as I did a similar thing with the `MissileFactory` [earlier](http://kenbonny.net/2016/09/21/unit-testing-part-4/). This means that the DI framework can replace some of my code. Less code means less maintenance and less chance for bugs.

Next I register all classes in the same assembly as the `WeaponsAssembly` class with the `IMissileFactory` interface. I added this class with this specific name to easily identify which assembly I am referencing. This means that if I resolve IMissileFactory, I will get an instance of MissileFactory. I could have also registered all classes in the `WeaponsAssembly` assembly according to their interfaces, but I haven't figured out how to register default instances or give certain instances names when mass registering them. Might be a fun experiment.

To get an instance of an interface, I ask the container to resolve one for me:

```
container.GetInstance<ILauncher>();
```

## One last test

There is just one problem that I'm seeing. When I comment out any line of code in the configuration, the compiler won't give me a warning that something isn't right. That's because problems with the DI will only become visible at runtime. So what do I do to remedy this? That's right: I write a unit test!

```
[Fact]
public IocConfigurationTests()
{
  container = Jarvis.Jarvis.ConfigureContainer();
}
public void Ioc_configuration_should_resolve_ILauncher()
{
  var instance = container.GetInstance<ILauncher>();
  Assert.NotNull(instance);
  Assert.IsType<Launcher>(instance);
}
```

This will let me know if there is ever a problem with resolving my most important types.

The last problem I encounter is that the `ConfigureContainer` method is internal. So, how can I access it in another project? In the `AssemblyInfo` class of the `IronMan.Jarvis` project, I add the line `[assembly: InternalsVisibleTo("IronMan.UnitTests")]`. This will let the `IronMan.UnitTests` project access anything with the `internal` access modifier in the `IronMan.Jarvis` project.

## Conclusion

With this, I hope to shed a little more light into the unit testing space. Not just how to write nice unit tests, but how to improve overall code by adding more structure and bringing it all together in a working application. Writing tests gives me a lot more confidence in the code I produce, results in a lot less bugs and grants me more time to work on new features because I have to revisit code less.

I also want to stress that a high level of good unit test coverage is not easily obtained. I took it step by step and increased my knowledge and proficiency with testing. I start with tests for the class that I'm currently working on, eventually I'll get to most classes because of refactoring/reviews. If I'm stuck, I reevaluate if my approach to the problem is correct and ask myself how I can improve my code so it's more easily testable.

Writing these articles and preparing this talk have helped me gain more insight into what unit testing is and how to improve my own code further. It cost me a lot of time, but I think it's been worth it.
