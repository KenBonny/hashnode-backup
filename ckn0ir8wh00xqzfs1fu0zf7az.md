---
title: "Complex object initialization"
datePublished: Mon Feb 06 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ir8wh00xqzfs1fu0zf7az
slug: complex-object-initialization
tags: csharp, dotnet, dotnetcore

---


[ReShaper](https://www.jetbrains.com/resharper/) just showed me an optimisation I did not know existed, there's another way to initialise objects.

I was aware of a few ways to initialise properties on an object. The first is creating an object and then setting its properties. The second is using object initialisation. When I want to set properties of a complex property, then I do it in the same way.

```
// first way
var obj = new MyObject();
obj.Property = "value";
// second way
var obj = new MyObject { Property = "value" };
// complex
var obj = new MyObect();
obj.Inner = New Inner();
obj.Inner.Property = "value";
// or
var obj = new MyObject { Inner = new Inner { Property = "value" } };
```

For complex objects, there is another way to set the inner property, but only if the object is already initialised in the constructor of the encapsulating object. That might sound complex, but it's quite simple. Let me demonstrate it in the `Outer` and `Inner` classes below. (These are not related to the previous examples.)

```
class Outer
{
  public Inner Initialized { get; set; }
  public Inner NotInitialized { get; set; }
  public Outer()
  {
    Initialized = new Inner();
  }
}
 
class Inner
{
  public string Value { get; set; }
}
```

There is an easy way to initialise the `Value` property of the `Inner` class when I initialise the object in the `Outer` class. The technique will work on the `Initialized` property, but not on the `NotInitialized` property.


```
new Outer
{
  Initialized = { Value = "will work" },
  NotInitialized = { Value = "NullReferenceException will be thrown" }
}
```

I have no idea how this technique is called or if it is something that is only available from C# 6 and later, but I found it pretty cool when I discovered it. If anybody knows how this feature is called, please [contact](http://kenbonny.net/about/) me and let me know.

Full code can be found in a [gist](https://gist.github.com/KenBonny/739acfdf5f76a811d2cf475fb7590046).
