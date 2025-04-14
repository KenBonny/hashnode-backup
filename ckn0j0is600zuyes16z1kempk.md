---
title: "Splitting IMediator interface"
datePublished: Mon Oct 05 2020 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j0is600zuyes16z1kempk
slug: splitting-imediator-interface
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381210677/c7Gynl8mi.jpeg
tags: csharp, dotnetcore

---


For a past project, I used the awesome [MediatR package](https://github.com/jbogard/MediatR) to send messages throughout the system. Because I used the pipeline for some compute heavy checks, it was not wise to send a request from a request handler. That is why I split the functionality for sending requests and notifications or events.

When a message entered the system, mostly via a REST endpoint, it got dispatched through MediatR to the corresponding handler. The message travels through the pipeline where some logging was done, some validity checks were performed and sometimes, there were even some security checks (e.g. "can this user access this data"). In this application, the pipeline is not the most lightweight part.

For some actions, I wanted to reuse logic from other handlers. Each handler has a single responsibility and I'd be reusing code, good decision in my opinion. Unfortunately, this triggered the pipeline each time which was not necessary at that point. I quickly saw that this significantly slowed the application. That is why the team and I decided to never send requests from within request handlers.

All said and done, we refactored this pattern of sending requests within handlers to simple service calls. That way, the reused request handler was nothing more than a facade in front of the service being called.

Notifications were also being used throughout the system to notify other parts when certain events happened. This would mean that the `IMediator` interface was passed into a significant number of handlers so they could publish these notifications (or events if you like that term better).

This also meant, that the team has easy access to the send request functionality. Now being the diligent programmers that we all are, I (and other team members, especially the newer ones) never succumbed to the temptation of cutting corners. So we always refactored the second handler into a service and called the functionality via the service. Or maybe not always...

Because that send request is just so easy to (mis)use, it still happened more than I would've liked. We all knew that not refactoring would just come to bite us later. From time to time, for whatever reason (pressure, tired, deadlines, new team member,...), it happened again.

That's when I created a specific interface for sending events through the system. I created an implementation that used the MediatR library. This allowed us to use the MediatR publishing mechanism, without exposing the send request functionality.

```
public interface IPublisher
{
  Task Publish<TNotification>(TNotification notification, CancellationToken cancellationToken = default)
    where TNotification : INotification;
}

public class MediatrPublisher : IPublisher
{
  private IMediator _mediator;
  public MediatrPublisher(IMediator mediator) => _mediator = mediator;
  public Task Publish<TNotification>(TNotification notification, CancellationToken token = default) => _mediator.Publish(notification, token);
}
```

Because I really like what [Jimmy Bogard](https://jimmybogard.com/) ([@jbogard](https://twitter.com/jbogard), the creator of the MediatR package) does, I've recently submitted a [PR](https://github.com/jbogard/MediatR/pull/563) to get this into the MediatR package. We all know it's much better to rely on somebody else's interface than to create our own (who noticed the sarcasm dripping from this sentence?).

In all seriousness, I think it will benefit the MediatR package to separate these concerns. That is why I've created two new interfaces: `IPublisher` and `ISender`. These contain the `Send` and `Publish` methods that resided in the `IMediator` interface. Because not everybody wants to switch to these specialised interfaces, I left the `IMediator` interface in place and have that inherit from the new ones.

```
public interface IPublisher
{
  Task Publish(object notification, CancellationToken cancellationToken = default);
  Task Publish<TNotification>(TNotification notification, CancellationToken cancellationToken = default)
    where TNotification : INotification;
}

public interface ISender
{
  Task<TResponse> Send<TResponse>(IRequest<TResponse> request, CancellationToken cancellationToken = default);
  Task<object?> Send(object request, CancellationToken cancellationToken = default);
}

public interface IMediator : ISender, IPublisher { }
```

I'm a big fan of Jimmy's work and I hope that with this change, I've helped improve the quality of life for a number of programmers, including mine. I'm not sure when this will be available in the MediatR package, but I hope soon.
