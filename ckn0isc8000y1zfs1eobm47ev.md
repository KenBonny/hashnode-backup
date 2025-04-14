---
title: "DateTimeOffset is the new DateTime"
datePublished: Mon Aug 01 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0isc8000y1zfs1eobm47ev
slug: datetimeoffset-is-the-new-datetime
tags: csharp, dotnet, dotnetcore

---


On my current work project, the system receives a document with dates and times in the local format (all Belgian local time). In the metadata however, we receive them in the UTC format. After a bit of processing, nobody knew which date was in what format anymore. Was the UTC time changed to local before saving it to the database? Or did we save the dates from the document as UTC. Why is it displayed differently on one part of the page compared to the next.

A solution was required and unfortunately we couldn't use the excellent [NodaTime](http://nodatime.org/) package from Jon Skeet due to our employer not trusting it. **Sigh**

After a bit of searching, I stumbled upon the [`DateTimeOffset`](https://msdn.microsoft.com/en-us/library/system.datetimeoffset.aspx) class. Microsoft should learn to name things correctly. When I first read the name `DateTimeOffset`, I thought this object stored only the offset for a `DateTime` object. Apparently it handles everything from dates and times with an offset according to the UTC standard. And it is native to the .NET framework. Basically how you expect the `DateTime` class to behave.

So we replaced most of the `DateTime` classes by the `DateTimeOffset` and all of our problems went away. The dates are now stored in the database as UTC time and when they are loaded, we can update them easily to the local time. The out of the box serialization includes the time offset (which the `DateTime` object ignores). This removed errors when serializing to JSON and passing it to JavaScript code.

In future projects, I will use `DateTimeOffset` for all my date and time needs instead of the `DateTime` class. Microsoft themselves even [encourage it](https://msdn.microsoft.com/en-us/library/bb384267(v=vs.110).aspx#Anchor_1). So I urge you to do the same.
