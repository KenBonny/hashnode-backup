---
title: "Fun with claims"
datePublished: Tue Jul 19 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0itoh400xjyes1hbnx3mek
slug: fun-with-claims
tags: csharp, security, dotnet, dotnetcore

---


For the project I'm working on, I have to grant users access to pages. Pretty standard, [ThinkTecture](http://www.thinktecture.com/identityserver) solved this problem long ago. Since this is a government project, they don't understand the benefit of using a lot of outside packages. So, based off of a blog post from Dominick Baier on [using claims based authorization in MVC](https://leastprivilege.com/2012/10/26/using-claims-based-authorization-in-mvc-and-web-api/), I build the following method:

```
public bool IsAllowedTo(string action, params string[] resources)
{
  if (principal == null)
  {
    return false;
  }
  var authorized = false;
  foreach (var claimType in this.resources
                .Select(x => string.Format("urn:{0}/{1}", this.action, x)))
  {
    var claim = principal.FindFirst(claimType);
    if (claim == null)
      continue;
    bool value;
    if (bool.TryParse(claim.Value, out value))
      authorized |= value;
  }
  return authorized;
}
```

This is being called by a custom authorization attribute that inherits from `AuthorizeAttribute` in the overriden `AuthorizeCore` method.

This allows me to have permissions such as in the blog post:

```
[MyAuthorization("Read", "ControllerAction", "MyView")]
public IActionResult ControllerAction()
```

This would mean that I would need a claim with the type "urn:Read/ControllerAction"or "urn:Read/MyView" with the value "true" to be able to view the page.

The ThinkTecture project goes a lot deeper and is a lot better, so use that if you can. If not, for the basic purpose here, this will check if a user has the correct claims.
