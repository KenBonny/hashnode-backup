---
title: "Instance method versus static method"
datePublished: Mon Oct 09 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iv5i000yjzfs1djmh63a9
slug: instance-method-versus-static-method
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617896804708/_fIy8TkiD.jpeg
tags: csharp, dotnet, dotnetcore

---


In the past week, some of my colleagues didn't understand difference between instance and static methods. They didn't know why the instance methods couldn't be called without calling `new` and why the static methods couldn't access instance fields. So if they didn't know, maybe some of you will find it enlightening as well.

Basically, it all boils down to memory. To help me understand how this works, I imagine memory as a huge closet with small compartments. Like the one in the top picture, but infinitely larger.

So how does this relate to the static and instance methods. When I write an object, lets say a simple `Person` class.

```
public class Person
{
  private string _name;
  private int _age;
  private static string _nameAge;
  public Person(string name, int age)
  {
    _name = name;
    _age = age;
  }
  public string GetNameAndAge()
  {
    return Concatenate(_name, _age);
  }
  public string CachedNameAndAge()
  {
    return _nameAge;
  }
  public static string Concatenate(string name, int age)
  {
    _nameAge = $"{name} {age}";
    return _nameAge;
  }
}
```

Then the code describes the way the application will behave during runtime. This code is actually the blueprint of what is being constructed in memory. When I use the `new` keyword (as in `new Person()`), an actual instance will be constructed in memory. This instance will be placed inside one compartment and can only access what is present in that compartment. That is how one instance doesn't use the values of variables of other instances.

The difference with static is that it only gets one memory compartment. This memory compartment will be allocated when an instance of the governing class is instantiated or any static field, property or method is called. Then the static constructor (`static ClassName()`) will be called. Keep in mind that a static constructor will only be called once during the execution of a program. If you want to know more about how static behaves, I found [this blog post by Jon Skeet](http://csharpindepth.com/Articles/General/Beforefieldinit.aspx) to be a good resource.

Lets clarify all this theoretical talk with an example. Lets start a new application, the memory would be empty, I'm ignoring all the .NET libraries that need to be loaded to make the application work.

![empty-memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380954316/cYimvuH1T.jpeg)

During the runtime, a new `Person` object is instantiated. The first thing that happens, is that the static `Person` object gets a memory compartment and the static constructor (if one is present) is called. So now we have the following memory "snapshot".

![static-memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380955791/rxhGfSeJg.jpeg)

So after the static instance is allocated, the instance of the class is created.

![single-instance-memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380957407/kudiw7NAk.jpeg)

When I call the `GetNameAndAge` method on the instance, it passes those values to the `Concatenate` method. This will set the `_nameAge` field to "_Ken 30_" and will return that value, which in turn will be passed back to the caller of the `GetNameAndAge` caller.

Let's complicate it a bit more and instantiate a second person object. The static instance is already instantiated, so it doesn't need to be created anymore (so no static constructor call). The second object is instantiated.

![multi-instance-memory.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380958941/LCSItplIr.jpeg)

Now the `GetNameAndAge`method on the second `Person` instance gets called. This forwards the call to the static `Concatenate` method, which overwrites the current value of `_nameAge` to "_John 20_" and returns that value back to the second instance.

Now it gets interesting. When the `CachedNameAge` on the first `Person` instance gets called, it will return the value of `_nameAge`. Which at this point is "_John 20_" and not "_Ken 30_" as the caller would expect.

This behaviour can be (ab)used to share information between two or more instances. I do not recommend using static fields or properties to send information of one instance to other instances. There are far more reliable forms of communication to do that, such as throwing and processing events.

With this image, I also hope to show why static methods cannot access instance fields, properties or methods. The static instance simply doesn't know to which instance they should be looking for values. Should the `Concatenate` method take the values from the first or second instance of the `Person` object. Maybe there are no instances because the `Concatenate` method is being called before any instance is created.

The reason why instance methods can access static fields, properties and methods is because a call to the static method `Concatenate` is replaced the longer `Person.Concatenate` call. This indicates where in memory the call should be redirected. All of this happens in the compiler and it knows where to look because it is defined in the same class.

For the readers who use [Resharper](https://www.jetbrains.com/resharper/), the reason why Resharper suggests some methods can be made static, is that it would require less memory. If a method doesn't need instance variables, then it is a lot more memory friendly to create it once in the static memory compartment. All instances then refer to the static method and it doesn't need to be duplicated. This lets the instances take up less memory because some code is shared. At the end of the day, this is just a micro-optimisation and is offset by how smart compilers have become.

What I want you to remember is that static members get their own memory space, separate of the object instances they are defined in. That is why instance variables cannot be accessed from static methods. Anybody using static fields or properties in instance methods should be very careful because other instances of the class can access the shared memory and change the values.

If you take all these things into consideration, static can be a very good addition to your toolbox when solving problems. Just remember to use static in moderation.
