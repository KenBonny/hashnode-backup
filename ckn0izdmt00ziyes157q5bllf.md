---
title: "Patterns and disciplines from a proof of concept"
datePublished: Mon Sep 11 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0izdmt00ziyes157q5bllf
slug: patterns-and-disciplines-from-a-proof-of-concept-part-5
tags: design-patterns, csharp, unit-testing, dotnet, dotnetcore

---


_While I worked at my previous employer, I build a proof of concept to improve their ability to search. I will rebuild that proof of concept and I'll highlight all the patterns and principles I used to build this code. All code related to this proof of concept can be found in a [repository](https://github.com/KenBonny/KenBonny.Search) on [my Github account](https://github.com/KenBonny)._

In this fifth and last part, I'll talk about additional benefits that this implementation can use.

Now that I established a decent base, lets see how this can be extended without much hassle. Throughout this article, I want to take a look how I segmented the code. In closing, I'll touch on the tests that I wrote for this proof of concept.

The first scenario is that when a seat for a diner has to be found, I first do a check that the diner hasn't reserved a seat already. To accomplish this, I applied the [decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern) on the `ISeatSearcher` interface. By both implementing and injecting the `ISeatSearcher` interface, I can place this decorator in between another searcher. Take a look at this example:

```
public class ReservationCheckerDecorator : ISeatSearcher
{
  private readonly ISeatSearcher _innerSeatSearcher;
  private readonly IReservationRepository _reservationRepository;
  public ReservationCheckerDecorator(ISeatSearcher innerSeatSearcher, IReservationRepository reservationRepository)
  {
    _innerSeatSearcher = innerSeatSearcher;
    _reservationRepository = reservationRepository;
  }
  public IReadOnlyCollection&lt;seat&gt; FindSeats(SearchQuery query)
  {
    var unreservedSeatForDinerQuery = query as UnreservedSeatForDinerQuery;
    if (unreservedSeatForDinerQuery == null)
    {
      return _innerSeatSearcher.FindSeats(query);
    }
    var reservedSeat = _reservationRepository.FindReservedSeat(unreservedSeatForDinerQuery.DinerFirstName, unreservedSeatForDinerQuery.DinerLastName);
    if (reservedSeat != null)
    {
      var returnSeat = new Seat(reservedSeat.Table.Section.Restaurant.Name,
        reservedSeat.Table.Section.Id,
        reservedSeat.Table.Id);
      return new[] {returnSeat};
    }
    return _innerSeatSearcher.FindSeats(query);
  }
}
```

By implementing the `ISeatSearcher` interface, I can pass it to any constructor that requires that interface. By injecting the `ISeatSearcher` interface, I can pass along an implementation that actually searches when no reservation is found.

The `IReservationRepository` interface allows me to make a very concise lookup in the database to look just for the diner, this allows the lookup to be very fast. If there is no reservation, then the next`ISeatSearcher` will be accessed to continue searching. If there is a reservation, then only the details of the reserved seat will be returned.

This code is placed in the _DefaultImplementation_ project. I put it there because the class puts itself in front of a search. I could put it in the _Core_ project so it can be used in all implementations, but the `ReservationCheckerDecorator` is meant to be placed in front of a `ConfigurableSeatSearcher`.

The second scenario is more technical in nature. When I was writing the tests, I noticed that if the number of `SearchQuery`s grows, it will be hard to know which combination of `IRestaurantRepository`s,  `IFilter`s, `IScoreCalculator`s and `ISorter`s are necessary to get the correct result. To keep track of which combination goes with which query, I implemented a simple [mediator pattern](https://en.wikipedia.org/wiki/Mediator_pattern). I could use an external library such as MediatR, but because all I needed was a way to route a `SearchQuery` to the accompanying `ISeatSearcher`, I build a simple implementation specifically for this scenario.

The mediator pattern is used to route commands (queries in this case), to the correct handler ( an `ISeatSearcher` in this case). So I first made a mechanism to register a `SearchQuery` and pass along the corresponding `ISeatSearcher` instance. For this, I used a `Dictionary&lt;Type, ISeatSearcher&gt;`. The dictionary is hidden inside the `SearchMediator` class, which has a `Register&lt;SearchQuery&gt;(ISeatSearcher instance)` method to indicate which instance of an `ISeatSearcher` should handle which `SearchQuery`.

It is meant to be instantiated in a higher level of the code, so this mediator can be used wherever the `ISeatSearcher` is requested. When the `FindSeat` method is called, the `SearchQuery` instance will then determine which actual seat searcher will be called internally. This way, it can be used seamlessly, just like the previous `ReservationCheckerDecorator`.

```
public class SearchMediator : ISeatSearcher
{
  private readonly Dictionary<Type, ISeatsearcher> _searchers = new Dictionary<Type, ISeatsearcher>();
  public void Register<T>(ISeatSearcher seatSearcher) where T : SearchQuery
  {
    var queryType = typeof(T);
    if (IsKnownType(queryType))
    {
      _searchers[queryType] = seatSearcher;
    }
    else
    {
      _searchers.Add(queryType, seatSearcher);
    }
  }
  public IReadOnlyCollection<ISeat> FindSeats(SearchQuery query)
  {
    var queryType = query.GetType();
    if (IsUnknownType(queryType))
    {
      throw new ArgumentOutOfRangeException("Unknown query type: " + queryType);
    }
    return _searchers[queryType].FindSeats(query);
  }
  private bool IsKnownType(Type queryType)
  {
    return _searchers.ContainsKey(queryType);
  }
  private bool IsUnknownType(Type queryType)
  {
    return !_searchers.ContainsKey(queryType);
  }
}
```

This is compatible with the `ReservationCheckerDecorator` class, because that implements the `ISeatSearcher` and can thus be registered. I could even put the decorator in front of the mediator if I want to check all searches for registrations. In my opinion, this is a very flexible system. [An example](https://github.com/KenBonny/KenBonny.Search/blob/master/KenBonny.Search.Tests/SearchUseCases.cs) of how this works can be found in the tests that accompany the code.

The `SearchMediator` code is located in the _Core_ project, because this has the potential to be used by multiple implementations of the `ISeatSearcher` interface. Thus, everywhere the interface is used, the mediator pattern should be available.

The last thing I want to focus on are the tests. I have 5 tests and together they cover roughly 85% of the codebase. That is quite a high coverage result for so little tests. That is because I don't test every little class. I haven't written tests for each filter, sorter, repository, etc. That would go into too much detail. Every time I refactor a class, one or more tests would break. That would cost me an immense amount of time to keep the tests code in sync with the production code.

What I did was write tests that describe use cases. The use cases don't change that often and when they do, it will take some work to rework them. The basic "search for seats" use case will stay the same for quite some time. So the tests don't need to change that much. I can refactor the code, change implementations and experiment with new ways of doing things and still be certain that the correct seat is being found.

A few tests that make sure that the majority of my code does what it's supposed to do, isn't that a great addition to the codebase.

With [these 5 articles](https://kenbonny.net/tag/poc-patterns-and-disciplines/), I hope to have given more insight in how I build a reusable and extendable component that can be easily switched out for testing or when another, more efficient way is found. Use interfaces so code talks against abstractions which are not only easily exchanged or reused, like the different filters and sorters, but can also be extended through decorators and other patterns. Don't forget that tests will save you a lot of headaches. Happy coding.
