---
title: "Contributing to AddFeatureFolders package"
datePublished: Mon Mar 12 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0is1wd00xyzfs19iduh783
slug: contributing-to-addfeaturefolders-package
tags: mvc, csharp, aspnet-core, dotnetcore

---


After I read the [MSDN article](https://msdn.microsoft.com/en-us/magazine/mt763233.aspx) about structuring an MVC app in feature folders, I wanted to create my own Nuget package or dotnet template to easily achieve this.

I started full of good intentions. After 4 weekends of trying stuff out, I stumbled upon [OdeToCodes AddFeatureFolders repository](https://github.com/OdeToCode/AddFeatureFolders). A little later I also found [his blog post](https://odetocode.com/blogs/scott/archive/2016/11/29/addfeaturefolders-and-usenodemodules-on-nuget-for-asp-net-core.aspx) about the package. He was leaps and bounds ahead of me in terms of setup.

With a single extension method `.AddFeatureFolders()` on an `IMvcBuilder`, he sets up the whole feature very easily and extremely flexible. Don't want to name the feature base folder "Features", that's possible. When some views are located in another folder, then there's an option to override the default view location (which is set to "{feature folder name}{controller name}{view name}.cshtml" by default).

The option that impressed me the most, is the ability to overwrite the default way of finding the feature folder name. The default looks at the namespace of the controller, finds the feature folder name and derives the path from the remainder of the namespace. For example: if I have a controller `KenBonny.Features.Orders.OrderController`, then it will look for the `Index` view at "Features\\Orders\\Index.cshtml".

The "Feature" folder will change if I specify a different one in the settings. So the `KenBonny.Foo.Bar.Administration.Overview.OverviewController` with a feature folder name of "Foo" will look for the Index view at "Foo\\Bar\\Administration\\Overview\\Index.cshtml". If this system does not fit my needs, I can always change the way the views are found.

What I added is support for Areas through a second extension method `.AddAreaFeatureFolders()`. If I want to group some functionality together, I can now do so via Areas. This lets me organise my site with urls like "domain.com/area/feature/action". For example, all administration can be grouped into "domain.com/admin/overview/index" or the whole purchasing department can have their own section in "domain.com/purchasing/orders/index".

The default areas folder is set to "Areas" to separate the area features from the regular features. This is configurable, so you can place the areas into the features folder by giving setting the `<span class="pl-en">AreaFolderName` to the same name as the feature folders.

Unfortunately, I require the `FeatureFolderName`. The default is set to "Features", just like the base functionality. If I don't change the name in for the feature folder setup, no setup is needed. I require this to know where the "Shared" folder will be located. This folder is always located under the "Feature\\Shared" folder. However, if I set a custom `FeatureFolderName`, then my web app needs to know where to look for the shared folder.

Lastly, I can add a `<span class="pl-en">DefaultAreaViewLocation`. This lets me add a location to check for views, if I want to add a special folder for areas. Just like in the feature folder functionality.

Using features has helped me group important functionality together. I hope this addition to this Nuget package will help others to better organise their code.
