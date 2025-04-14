---
title: "Correctly unsubscribe in Angular"
datePublished: Mon Apr 23 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0is5fc00xzzfs19a1s9xwg
slug: correctly-unsubscribe-in-angular
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380820376/aF3naeqaS.jpeg
tags: web-development, angular, typescript

---


At work, a number of angular web pages were sending an increasing number of requests to the back-end. Apparently, if I reuse components that don't properly dispose of their observable subscriptions, they keep sending and processing multiple requests.

Imagine I have a page with multiple tabs that show the same type of data, lets say person information, and each tab displays info on different persons. On tab of Person 1, one request to https://example.com/data/person/1. When I navigate to the tab of Person 2, I notice 2 requests to https://example.com/data/person/2. Each tab navigation increases the number of requests by 1.

There is a service with a method that returns an `Observable`, for example:

```
@Injectable()
export class DataService {
  public getData(): Observable {
    const url = 'https://example.com/data/example';
    return this.httpClient.get(url);
  }
}
```

When the service gets injected and the `getData()` method is called, then I subscribe and store the result in a variable.

```
private loadData(): void {
  this.dataService.getData()
        .subscribe(data => { this.allData = data; },
            error => {
              console.error(error);
              this.allData = [];
            });
}
```

The problem that arises here is that the injected service with the `getData()` lives longer than the component it is injected into. So the subscription is still active when the component is destroyed. When the service is injected into another component, another subscription is added to the same endpoint. It can happen that instead of one call to [https://example.com/data/endpoint,](https://example.com/data/endpoint,) there will be several calls. One for each registered subscription.

There are three solutions according to [this article](http://brianflove.com/2016/12/11/anguar-2-unsubscribe-observables/). The general angular community likes this one the best: use `takeWhile`.

Upon subscribing, use `takeWhile` to register a boolean expression or value. When the boolean is set to false, the subscription will be stopped. The `OnDestroy` method will be the place where the boolean value is flipped.

```
export class SomeComponent implements AfterViewInit, OnDestroy {
  private keepReceiving = true;
  ngAfterViewInit() { this.loadData(); }
  ngOnDestroy() { this.keepReceiving = false; }
  private loadData(): void {
  this.dataService.getData()
                  .takeWhile(this.keepReceiving)
                  .subscribe(data => { this.allData = data; },
                             error => {
                               console.error(error);
                               this.allData = [];
                             });
  }
}
```

Now with the `takeWhile` boolean registered, everything will be cleaned up nicely. This memory leak is plugged and the performance is much better. Improvements all around.

**Update:** A friend of mine pointed out that the preferred way of unsubscribing is through the [AsyncPipe](https://angular.io/api/common/AsyncPipe). I didn't fully understand how to use the async pipe, but my friend gave a brief, yet thorough understanding. It basically boils down to that the async pipe will unsubscribe all subscriptions for you. Somehow, it works out from what you passed in what to unsubscribe. The updated code should look something like:

```
private loadData(): void {
  this.allData = this.dataService.getData()
                     .pipe(map((state) => state));
}
```

And in the front-end, I can use the data just like I want. All I have to do, is put a pipe with the `async` keyword behind it.

```
app-my-component data="allData | async"
<div></div>
<div>{{allData.text | async}}</div>
```

Now Angular will take care of all the heavy lifting of subscribing and unsubscribing. A big thanks to my friend (who likes to remain anonymous).
