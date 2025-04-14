---
title: "Static should be used sparingly"
datePublished: Mon Jan 07 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j0mw600zxyes149f55zfu
slug: static-should-be-used-sparingly
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617802477150/3t1e1dwtN.png
tags: csharp, dotnet, dotnetcore

---


New year, new blog posts! Lets start with a problem from work. Let me present you with the problem code I had to analyse.

Once or twice a month, the name "Ken" got written to the database and another name was written to the log. This is a reproduction from the ASP.NET web call, actual code was a lot more complex.

```
private static string Name;
public static void Execute() 
{
  Name = Cache.GetNameFromSession();
  var description = $"I'm using the name {Name}";
  SaveToDatabase(description);
  Logger.Info($"Saving Name {Name} to the database");
}
```

If you figured out what went wrong, congratulations. I had to look an embarrassing amount of time to spot the `static` in the field description. After I noticed that, I recognised the timing problem that can occur.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381214903/lNmVtN_G4.jpeg)

Due to the static nature of the field, it is shared between multiple requests. The first request sets the `Name` to "Ken", which creates the description "I'm using the name Ken" and saves that to the database. When it's time to log the name, the second request has updated it to "Sophie" which is what will get logged.

Fortunately, the solution is quite simple: make the field non-static so it's bound to the instance, then each request will have their own instance. The method itself is static and I couldn't change that, so I cannot make the field non-static. The next solution is to call the cache where it needs to be called. A local variable is also not shared between multiple requests, so the problem doesn't occur anymore.

```
public static void Execute() 
{
  var name = Cache.GetNameFromSession();
  var description = $"I'm using the name {name}";
  SaveToDatabase(description);
  Logger.Info($"Saving Name {name} to the database");
}
```

Let this be a lesson for all of us: use `static` sparingly. It's great for a factory method such as `Person.Create(name, age)`, but it's the cause of a lot of subtle bugs when used in concurrent environments such as a web server.
