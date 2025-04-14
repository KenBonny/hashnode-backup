---
title: "Introducing C# 10"
seoTitle: "Introducing csharp 10"
datePublished: Fri Apr 30 2021 18:42:43 GMT+0000 (Coordinated Universal Time)
cuid: cko4nyhuv033i94s18xdq0kkj
slug: introducing-csharp-10
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1619808066369/NL2jGN8Ih.png
tags: csharp, dotnet, dotnetcore

---

> Here is a first for my blog: @[Rwing](@Rwing) offered to translate it to Chinese. So I took him up on it and you can read his translation on [his blog cnblogs.com](https://www.cnblogs.com/Rwing/p/introducing-csharp-10.html). I hope it says nice things about me. 😁

# C# 10

Earlier this week, I followed a [talk](https://www.youtube.com/channel/UCU0f_2rwIlvLC35GfGk_8kg) by [Mads Torgersen](https://twitter.com/MadsTorgersen) at [DotNet SouthWest](https://www.meetup.com/dotnetsouthwest/), he's the lead designer for the C# language at Microsoft. He outlined the cool new things C# 10 will contain. Let's take a quick look at some of the good things coming our way!

> Small disclaimer, most of these changes are pretty much done as he was clearly showing off. Since it's still in active development, I can't guarantee that everything will be exactly as is when C# 10 releases.

The first thing he talked about, is how the current implementation of `record` uses a `class` (read: reference type) as the base object. There will also be a `record struct` variant available so the underlying type can be a value type. The difference is that a regular `record` will pass from function to function by reference and a `record struct` will be copied by its values. The `record struct` will include `with` support.

At the same time, it will be possible to add operators to `record`s. It will be available for both `record` types.

```
record Person(string Name, string Email)
{
  public static Person operator +(Person first, Person second)
  {
    // logic goes here
  }
}
```

One of the goals the C#-team is focussing on, is making initialisation of objects easier. That is why it will be possible to flag properties of a `class`, `struct`, `record` or `record struct` as `required`. It makes those properties mandatory to fill in. This can be done via a constructor, or this can be done with object initialisation. The two class definitions below are equivalent. If you write it with the `required` keyword, you cannot instantiate the Person without setting the `Name` property. The compiler will throw errors and fail to compile.

```
class Person
{
  public required string Name { get; set; }
  public DateTime DateOfBirth { get; set; }
}

class Person
{
  public Person(string name) => Name = name;

  public string Name { get; set; }
  public DateTime DateOfBirth { get; set; }
}
```

To further improve properties, it will be possible to get rid of backing fields alltogether. The new keyword `field` will provide access to said backing field. This will be available for both setters as `init` only properties.

```
class Person
{
  public string Name { get; init => field = value.Trim(); }
  public DateTime DateOfBirth { get; set => field = value.Date; }
}
```

There will be a few nifty little enhancements in the next version as well. One is that the `with` operator will support anonymous types as well.

```
var foo = new
{
  Name = "Foo",
  Email = "foo@mail.com"
};
var bar = foo with {Name = "Bar"};
```

It will now be possible to create a single file with namespace imports that are used everywhere. For example, if there is a popular namespace that is used in virtually every file, say `Microsoft.Extensions.Logging.ILogger`, then it will be possible to add a `global using Microsoft.Extensions.Logging.ILogger` to any *.cs* file (I suggest the *Program.cs* or a dedicated *Imports.cs*) and the logger interface will be available in the entire project. Not the solution! Nobody can predict which imports are needed anywhere, so they are grouped per project.

Subsequently, there will also be an optimisation to `namespace`s. Now a `namespace` requires curly braces \{\} to group code which entails that all code will at least be indented once. To save that tab (or four spaces, whatever, you choose) and the screen real estate, adding a `namespace` anywhere in a file, will make all code belong to that `namespace`. Research has shown that nearly all code in one file belongs to the same `namespace`. The subsequent reduction in file size because all those tabs (or spaces) are saved might not be significant for one solution (even if it contains thousands of files), but on the scale of GitHub/GitLab/BitBucket/..., I think it will save them some space. Should anybody still want to include multiple namespaces in one file, the option to use the curly braces is still available.

```
// LegacyNamespace.cs
namespace LegacyNamespace
{
  class Foo
  {
    // legacy code goes here
  }
}

// SimplifiedNamespace.cs
namespace SimplifiedNamespace;
class Bar
{
  // awesome code goes here
}
```

There are some cool updates to lambda statements coming too. The compiler will have better support for inferring lambda signatures and it will be possible to add attributes as well. It will be possible to specify explicit return types to help the compiler understand the lambda.

```
var f = Console.WriteLine;
var f = x => x; // inferring the return type
var f = (string x) => x; // inferring the signature
var f = [NotNull] x => x; // adding attributes on parameters
var f = [NotNull] (int x) => x;
var f = [return: NotNull] static x => x; // adding attribute for a return type
var f = T () => default; // explicit return type
var f = ref int (ref int x) => ref x; // using ref on structs
var f = int (x) => x; // explicitly specifying the return type of an implicit input
var f = static void (_) => Console.Write("Help");
```

> Thanks at [Schooley](https://www.reddit.com/user/schooley/) for suggesting a less confusing example attribute!

Lastly, it will be possible to specify static methods and properties on interfaces. I know this will be a controversial topic, just like adding default implementations to interface. I'm not a fan of that last one. This however, could be very interesting. Imagine that you could specify the default value of an interface or specify creation methods.

```
interface IFoo
{
  static IFoo Empty { get; }
  static operator +(IFoo first, IFoo second);
}
class Foo : IFoo
{
  public static IFoo Empty => new Foo();
  public static operator +(IFoo first, IFoo second) => /* do calculation here */;
}
```

Personally, I like these changes. My favourite ones are the changes to `namespace` and the improvements to the interfaces. Anyway, the future is seeing sharp. Eh Eh...

I'll see myself out now.