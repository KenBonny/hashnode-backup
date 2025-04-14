---
title: "Use claims to authorise users to access specific data"
datePublished: Mon Nov 20 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j3fpe010tyes15hzpee3h
slug: use-claims-to-authorise-users-to-access-specific-data
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381346741/ERrPDOvZk.jpeg
tags: csharp, security, dotnetcore

---


The software I'm working on needs a new authorisation system. The system needs to be prepared for 3 scenarios: to restrict access to a page, to hide part of a page and to block access to data. Let's solve these problems using claims.

Let me talk a little bit more about the problem. I will be talking about a hypothetical situation as I don't want to talk about the actual business of the company I work for. I wouldn't like to disclose company secrets by accident. So lets say that the company I work for buys and sells houses.

The problem is that not every salesperson can buy or sell every house in the database.  They can only operate on specific houses, namely, the ones who are located in the country they live in.

In this little proof of concept, I'm going to build a solution that takes the individual house ids into consideration. I trust that it is easy enough to verify another field with the same techniques.

The first requirement is that we need to restrict a user to certain pages, this can be done very easily on the controller with the `Authorize` attribute. It is very easy to use and there are a lot of [tutorials](https://msdn.microsoft.com/en-us/library/system.web.mvc.authorizeattribute(v=vs.118).aspx) that [explain](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/roles) [how to use](https://msdn.microsoft.com/en-us/magazine/mt826337.aspx) [this attribute](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/secure-data) [already](http://docs.identityserver.io/en/release/index.html), so I'm not going to rehash those here.

The same goes for the second requirement. To hide certain parts of a page, I can use the [`IAuthorizationService`](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/views?tabs=aspnetcore2x) or the extension [tag](https://aspnetmonsters.com/2017/11/2017-11-05-authorize-tag-helper/) [helpers](https://aspnetmonsters.com/2017/11/monsters-weekly/ep111/). They too already have a lot of blog posts about them, so be sure to check those out.

What I want to focus on, is the data restrictions using claims. Maybe there are easier ways to limit access to data, I haven't found them yet. I also wanted to see how easy it would be to use claims to do this.

To try this out, I have created a small [asp.net core 2 application](https://github.com/KenBonny/ClaimsDataAuthorizationExperiment) that incorporates the page access and data access restrictions.

The whole application runs in memory, so if you download it, compile it and run it, it will always ask you to create a new user. Since I was experimenting, I added some password rules. As a result, you need a password that includes uppercase, lowercase, numbers, a special character and is between 6 and 100 characters long. Don't stress to much about it, once you stop running the app, everything is forgotten.

When you log on, the first view you see is that you do not have access to the home page. This is where the page restrictions come into play.

![no-access](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381340444/er4kz6x2Z.jpeg)

It's a little basic, but it does the job. Above the `PropertyController`, there is an `[Authorize("Property")]` attribute that checks if the user has the "Property" claim. If the user does not have this claim, this page will be shown.

I added a link in the navigation bar "ToggleAccessProperty" that will add the claim to the users claims and redirect back to the `PropertyController` index page. Now I get to see the property page, but it's not showing any data yet.

![no-properties](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381341915/M1s4QMQi8.jpeg)

This is because I have no access rights to any data yet. To fix this, let's go to the claims page and add two claims. One with type "seller" and value 1 and another with type "buyer" and value 2.

![add-claims](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381343519/uZmwj41s_.jpeg)

Now return to the properties page and see that you can buy and sell a house.

![properties](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381345218/lpUAmQf5A.jpeg)

Unlike the song, it's not some kind of magic. Before I show how I implemented the properties page, I'm going to back up a moment and reveal how I implemented the claims page.

The claims page is a part of the `AuthorizationController`, it has a page that returns all claims a user has.

```
[HttpGet]
public IActionResult Claims()
{
  return View(User.Claims);
}
```

The `AuthorizationController` also has 2 actions that add and remove claims.

```
[HttpGet]
public async Task&amp;lt;IActionResult&amp;gt; RemoveClaim(string type, string value)
{
  var user = await _userManager.GetUserAsync(User);
  var claim = new Claim(type, value ?? string.Empty);
  await _userManager.RemoveClaimAsync(user, claim);
  await _signInManager.RefreshSignInAsync(user);
  return RedirectToAction("Claims");
}
[HttpPost]
[ValidateAntiForgeryToken]
public async Task&amp;lt;IActionResult&amp;gt; AddClaim(string type, string value)
{
  var user = await _userManager.GetUserAsync(User);
  var claim = new Claim(type, value ?? string.Empty);
  await _userManager.AddClaimAsync(user, claim);
  await _signInManager.RefreshSignInAsync(user);
  return RedirectToAction("Claims");
}
```

Note here that I use the `UserManager` to add or remove the claims. The problem that I noted is that the claims didn't update automatically on the logged in user. That is why I call the `RefreshSignInAsync` method on the `SignInManager` with the current user. Otherwise I would need to log in and out to refresh the claims on my user. Take this into account when you are updating claims on other users as well and test this thoroughly.

Now that I can add and remove claims on the fly, lets look at the `PropertyController`. I kept this controller simplistic because I want to focus on the data, but know that you can move this functionality to a separate service or do some of this filtering in a database if you pass along the claim data.

The `PropertyController` only has one action, the `Index` action. In this I access a private list with 5 properties that can be either sold or bought. In reality these will come from a database with separate tables for properties that can be bought and sold.

```
public IActionResult Index()
{
  var availableProperty = new AvailableProperty
  {
    // User property can check relevant claims
    CanBuy = _properties.Where(User.CanBuyProperty),
    // threads current principal can also check the relevant claims
    CanSell = _properties.Where(_injectedUser.CanSellProperty)
  };
  return View(availableProperty);
}
```

In the `Index` action, I load the properties into a model that checks whether the property can be bought or sold by the currently logged in user. I added an [extension method](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) on the `ClaimsPrincipal` class that will perform the check.

```
public static bool CanBuyProperty(this ClaimsPrincipal user, Property property)
{
  return user.HasClaim("buyer", property.Id.ToString());
}
public static bool CanSellProperty(this ClaimsPrincipal user, Property property)
{
  return user.HasClaim("seller", property.Id.ToString());
}
```

At first, I had this functionality in the `PropertyController`'s `Index` action, but since controllers are just a thin layer to orchestrate what should happen, I did not deem this the right place to put this logic. That's why I made the extension method.

I also added two other extension methods that return the list of identifiers which property a user can buy or sell. This allows me to pass this information to a query and do the filtering in the database.

```
public static IReadOnlyCollection&lt;int&gt; BuyableProperties(this ClaimsPrincipal user)
{
  return user.Claims.Where(x =&gt; x.Type == "buyer").Select(x =&gt; x.Value).Select(int.Parse).ToList();
}
public static IReadOnlyCollection&lt;int&gt; SellableProperties(this ClaimsPrincipal user)
{
  return user.Claims.Where(x =&gt; x.Type == "seller").Select(x =&gt; x.Value).Select(int.Parse).ToList();
}
```

Now lets take another look at that `Index` action.

```
public IActionResult Index()
{
  var availableProperty = new AvailableProperty
  {
    // User property can check relevant claims
    CanBuy = _properties.Where(User.CanBuyProperty),
    // threads current principal can also check the relevant claims
    CanSell = _properties.Where(_injectedUser.CanSellProperty)
  };
  return View(availableProperty);
}
```

The `CanBuy` property is using the `User` property that is available in controllers. The `CanSell` property is using an `_injectedUser` field. Weird, isn't it.

This is exactly the same object, only the `_injectedUser` comes from a field that is injected into the controller. This allows me to inject the logged in `ClaimsPrincipal` object into services via dependency injection. I stumbled on this after I tried to use the `Thread.CurrentPrincipal` and it came back `null`.

In .net core, the `Thread.CurrentPrincipal` isn't being used anymore, since public static state can be manipulated by everybody (including third party libraries and frameworks) and is thus not secure.

To inject the `ClaimsPrincipal`, add this configuration to the `ConfigureServices` method in the `Startup` class or your dependency injection framework of your choice.

```
services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
services.AddTransient<ClaimsPrincipal>(provider => provider.GetService<IHttpContextAccessor>().HttpContext.User);
```

For this last part, I cannot take credit. I found the information in a [blog post from David Pine](http://davidpine.net/blog/principal-architecture-changes/).

This is how I would tackle the problem of authorisation of data to users with claims. If you know a better way of doing this, please get in touch with me via one of the [channels on my contact page](http://kenbonny.net/about/).
