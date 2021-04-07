---
layout: post
title: Reactive Programming with RxJS
tags: [reactive, rxjs, angular]
excerpt_separator: <!--more-->
author:  docHologram
---

The term "User Experience" is one of the most clichéd phrases in the world of software, one that is easily tossed around and becomes the linchpin of meaning for everything that we do for an application. Should we stagnate an infinite scroll to load more results or page them instead? I don't know. Which is a better user experience? Why is the button green? It gives the person a sense of achievement, therefore enhancing the user experience. Why are we using wingdings for a font? It enhances the user experience for our target audience, which is R2-D2. You get the point.

You could say that user experience is the _reason_ why we make the decisions we do when building our application. At the heart of the matter is how a person _feels_ when they use it. Destructuring user experience into it's various components of study such as psychology, color theory and haptics (or touch interactions) just name a few, all contain the underlying goal of ensuring that a person _feels_ pleasant when they use it. This post is concerned with another component of user experience... which is speed. When your application is able to respond quickly to state changes, we also contribute to a great user experience and this is how our application becomes _reactive_.

## So, what is Reactive Programming?

According to the homepage for [ReactiveX](http://reactivex.io/), the definitive source for all things Reactive Programming, reactive programming is a combination of aspects in the Observer and Iterator patterns with functional programming techniques. Another way to explain this is to think of _observing_ an asynchronous source of data and _iterating_ over that data with a pipeline of operations to map it, filter it, accumulate totals, etc. before displaying the results. Functional programming is the engine of that pipeline of operations in that the result from one operation is passed to the next operation in the pipeline.

## How Programming vs What Programming

Before we go further, let's also talk about imperative vs. declarative programming. This concerns the level of granularity in explain what the code is doing. The imperative approach is much more verbose in explaining what is going on under the hood, while the declarative approach obfuscates much of the nuts and bolts favoring a cleaner description of the code's intention.

Or as [a wiser man that I once said](https://ui.dev/imperative-vs-declarative-programming/):

> “You know, imperative programming is like how you do something, and declarative programming is more like what you do, or something.”
>
> - Tyler McGinnis

As you will see later on in this post, reactive programming is not only functional in style but also declarative in style as well.

Let's take a look at a side-by-side example.

![Imperative vs Declarative Approach](/assets/img/imperative-vs-declarative.png)

In both approaches, we initialize an array of 12 "raw" donuts straight off the assembly line. With the imperative approach, our code documents that we are going to iterate over each donut, determine if it is broken, add sprinkles if it is not broken and then add it to the box of sprinkledDonuts. With the declarative approach, we obfuscate the iteration of the for-loop by _implying_ it with our use of the chained filter and map methods. We can look at our declarative statement and read it as "box up the raw donuts that aren't broken and add sprinkles on them".

These two examples are written in Javascript, but you can see how the declarative approach resembles a pipeline with the filter and map methods. Keep this in mind as we head into authoring reactive programming.

## The Trinity of Reactive Programming

* Observables: receiving from a data source
* Operators: pipeline of actions on the data
* Subscriptions: consuming the final result

### Observables

Observables are the heart of the reactive programming paradigm in that we cannot have reactive programming without reading from a data source of some type. We can think of the donut assembly line in our previous example as an observable. Another example of an observable would be a social media feed that is constantly receiving new posts for display on the screen.

![Observables](/assets/img/observables.png)

### Operators

Operators form a pipeline of actions, with each operator passing it's result to the next operator in the pipeline. In the following example, think of beltOfDonuts as an observable:

```
beltOfDonuts()
    .pipe(
        filter(donut => donut.isNotBroken()),
        map(donut => donut.addSprinkles()),
        take(12),
        toArray()
    );
```

By the way, the result of each of these operators is itself an observable.

We aren't done quite yet. Ultimately, no one can eat these donuts with a subscription.

### Subscriptions

Subscriptions consume the end result, often called an emission. In fact, without a subscription our application never sees the result of the beltOfDonuts observable pipeline. Let's add a subscription to beltOfDonuts:

```
beltOfDonuts()
    .pipe(
        filter(donut => donut.isNotBroken()),
        map(donut => donut.addSprinkles()),
        take(12),
        toArray()
    )
    .subscribe((boxOfDonuts: Donut[]) => {
        const myDonuts = boxOfDonuts;
    });
```

In this example, we will continue getting boxes of 12 sprinkled donuts for as long as beltOfDonuts keeps giving them to us. Many observables have an onCompleted method that tells the pipeline and subscription that there is no more emissions. However, there are also many observables that do not have an onCompleted method and will keep the faucet on ad infinitum. This could cause memory leaks or unwanted side effects in our application if we leave a bunch of unused subscriptions lying around. Luckily, we can easily unsubscribe to these when we are finished:

```
const donutSubscription = beltOfDonuts()
    .pipe(
        filter(donut => donut.isNotBroken()),
        map(donut => donut.addSprinkles()),
        take(12),
        toArray()
    )
    .subscribe((boxOfDonuts: Donut[]) => {
        const myDonuts = boxOfDonuts;
    });

donutSubscription.unsubscribe();
```

Next, let's take a closer look at the three possible outcomes of any given observable emission:

### Observables Have a Trinity Too!

* onNext
* onError
* onCompletion

In our previous example, we only show what happens onNext which is we get a box of donuts. 

```
.subscribe((boxOfDonuts: Donut[]) => {
    const myDonuts = boxOfDonuts;
});
```

But what happens if something goes wrong like the donut belt stops working or the guy operating it says "take this job and shove it" and shuts it down? Well, we need to handle these scenarios, right?

We can supplement our subscription callback, the onNext, with an optional onError path and also an onComplete path if we want something to happen when and if the observable completes:

```
.subscribe(
    (boxOfDonuts: Donut[]) => {
        const myDonuts = boxOfDonuts;
    },
    (error: string) => {
        displayMessage(error);
    },
    () => {
        displayMessage('We are all done for the day!');
    }    
);
```

## Recap

OK, let's summarize what we have so far. 

1. Observables listen in on a data stream of some sort and emit values
2. Operators perform actions on that data returning their result as another observable to the next operator in the pipeline
3. Subscriptions consume the end result

## Reactive Resources

Thus far we have explored reactive programming in the context of JavaScript ([TypeScript](https://www.typescriptlang.org/) in particular, actually), but reactive libraries exist for all sorts of programming languages. Remember ReactiveX that I spoke of earlier? Well, in addition to providing a ton of explanatory docs and tutorials, there is also a [listing of languages](http://reactivex.io/languages.html) with Rx support such as:

* Java (RxJava)
* C# (Rx.NET)
* C++ (RxCpp)
* PHP (RxPHP)

...just to name a few. What we've been working with so far is [RxJS](https://rxjs.dev/guide/overview).

## RxJS Reiterated

Let's take a look an enhanced version of our beltOfDonuts example that does a better job of emulating a subscription that receives a box of 12 donuts.

```
beltOfDonuts()
    .pipe(
        filter(donut => donut.isNotBroken()),
        map(donut => donut.addSprinkles()),
        scan((donuts, donut, index) => (index % 12) ? donuts.concat(donut) : [donut]),
        filter(donuts => donuts.length === 12)
    )
    .subscribe(
        (boxOfDonuts: Donut[]) => {
            placeOnShelf(boxOfDonuts);
        }),
        (error: string) => {
            displayMessage(error);
        }),
        () => {
            displayMessage('We are all done for the day!');
        }
    );
```

In this example, we scrapped the take(12) operator for a scan operator instead. If you recall from earlier, take(12) will take 12 donuts and complete. Therefore, our subscription will only receive a single box of donuts. The use of scan here:

```
scan((donuts, donut, index) => (index % 12) ? donuts.concat(donut) : [donut])
```

works much like a reducer in that it will accumulate donuts until it gets to 12 (`index % 12`) and then initialize a new box with a single donut in it. We are using a bit of digital smoke and mirrors here because scan will emit a box with one donut, then a box with two donuts, _10 passes later_... then a box with twelve donuts, then a box with one donut... and so on. To emulate that our subscription only gets a box of 12 donuts, we utilize the filter operator, which will discard any box that has less than 12 donuts. Blasphemy, I know!:

```
filter(donuts => donuts.length === 12)
```

There are other operators in the RxJS library. Would you like to see them? OK, you asked for it...

## Gimme Some More... Operators

Here ya go; 105 of them last time I checked. 

![All RxJS Operators](/assets/img/all-operators.jpg)

In Lighthouse, we use a handful on a regular basis and maybe 15 of these operators overall. However, we'll take a closer look at a few observables in greater detail. Each of the following sections features a screenshot from [RxJS Marbles](https://rxmarbles.com/), which provides interactive marble diagrams illustrating what happens over time as operators emit values.

### startWith

![startWith](/assets/img/startWith.jpg)

There are scenarios where there is simply no emission from an observable but we need the reacting code to execute immediately. This could happen if the observable is listening to user events. If we have an address form with a dropdown of states or provinces whose list is dependent on the dropdown of countries that precedes it, we may want to default the country to the predominate country where this application is used. If we have an observable on the country select event that subsequently populates the states/provinces dropdown, then we can `startWith('US')` and populate the states dropdown with U.S. states immediately.

### take

![take](/assets/img/take.jpg)

We got a taste of the `take` operator earlier with our first iteration of `beltOfDonuts`. This one is pretty straight-forward in that it will "take" x number of emissions and then complete the observable, ignoring any subsequent emissions. There are a few other operators analogous to take. If you don't want to set a static number of emissions to take but would rather utilize some condition to predicate the completion of the observable, you could use `takeUntil` or `takeWhile`.

```
takeUntil(() => customerOrderFulfilled)

takeWhile(() => !customerOrderFulfilled)
```

There is also the family of operators that serve as the inverse of take. You may want to `skip` x number of emissions before you "take" them or `skipWhile` some condition is not fulfilled or `skipUntil` some condition is fulfilled.

### scan

![scan](/assets/img/take.jpg)

The scan operator will accumulate the values from another observable's emissions and spit that out as it's own emission. This is essentially a reducer function for reactive programming.

A wise singer once said that we're never gonna survive unless we are a little crazy. So let's get crazy and put an observable inside another observable.

## Observable in Another Observable

![Say Whaaat!](/assets/img/say-whaaat.jpg)

So far, we have looked at a sequential pipeline of operators where each operator pays no mind to what the previous operator did and simply does it's job. Yet, there are circumstances in which an observable's own behavior is affected by the result of another observable. We may want the value of both observables. In these scenarios we can say that there is an observable _inside_ of another observable. Pretty weird, huh? Let's try and clear this up by looking at 3 types of mapping operators. These are `mergeMap`, `concatMap` and `switchMap`.

### concatMap

![concatMap!](/assets/img/concatMap.jpg)

If we imagine that an outer observables emits plain donuts and the inner observable sprinkles each donut, then a concatMap here would wait until all of the donuts were emitted and then it would sprinkle them all before emitting each of the sprinkled donuts.

### mergeMap

![mergeMap!](/assets/img/mergeMap.jpg)

If we again imagine that an outer observables emits plain donuts and the inner observable sprinkles each donut, then a mergeMap here would sprinkle each donut as it came through and emit it.

### switchMap 

![switchMap!](/assets/img/switchMap.jpg)

Now switchMap is a little different in that an emission from the outer observable will stop the operation of the inner observable in order to "switch" to the new outer observable value. This is useful regarding http requests where we want to cancel the current request because a parameter of the request has changed. Think of a search engine input field that sends a request to search the internet for "dog" articles but the user wasn't done typing and really wanted "dogstar" articles instead (foreshadowing :wink:). Using a switchMap, we can stop the request for "dog" stuff and run a request for "dogstar" stuff instead.

Still into the donut examples? 

Imagine that our inner donut sprinkler observable determines which kind of sprinkles to drop based on the color of icing of the donut from the outer observable. If it was dropping chocolate sprinkles on the vanilla donuts before, but now chocolate donuts come through the pipeline, then _switch_ to vanilla sprinkles instead.

At the end of this article, I show you a search engine typeahead that utilizes a switchMap. It is an [Angular](https://angular.io/) application and there is a reason for that. 

## Angular is a front-end framework that is reactive by design

The Angular framework utilizes RxJS quite extensively and, in fact, RxJS is baked right into a new applications if you spin one up using the [Angular CLI](https://angular.io/cli). You can see it listed in the dependencies of the package.json:

{% include embed_stackblitz.html stackblitz_url='https://stackblitz.com/edit/angular-ivy-xp7ump?file=package.json' %}

Angular utilizes the concept of reactive forms, wherein you can easily hook into an observable of value changes to the entire form or to a group of form controls, or to a single control itself and do something reactively like check validity or update some other value on the screen.

Another interesting aspect of Angular is how http requests are wrapped in observables. Typically, we think of http requests as more like promises such that the request either succeeds or it fails and we will handle either appropriately. When we turn our requests into observables though, we can make use of our operators to transform the data before it reaches the calling component or we can catch errors and resubmit the request to try again.

Now, let's check out that typeahead search which goes by the cheeky name of Stellar Search!

## Stellar Search: Reactive Search Engine Requestor

So like I stated before, Stellar Search is an Angular application. The goal here is to illustrate not only the capabilities of observables but also their ease of use once you understand them. It's the "understanding" part that is difficult. I'd say it took me a solid year of using observables before I really started to "get it". You can [check out the Stellar Search repo here](https://github.com/docHologram/stellar-search).

We are mostly concerned with two observables. One is a pipeline of operators stemming from the keyup event on the search field so that every time the user adds a character, a new emission of the current value of the search field kicks off the pipeline. The other observable concerns the http request to Google where we map the results of the request to something more easily consumed by the requestor. We'll also see how these two observables are unified by the use of a switchMap operator.

Let's take a look at the search field observable first:

```ts
fromEvent(this.searchInput.nativeElement, 'keyup')
    .pipe(
        debounceTime(300),
        map((event: any) => event.target.value),
        filter((searchTerm: string) => searchTerm.length > 2),
        distinctUntilChanged(),
        tap((searchTerm: string) => this.searchTerms.push(searchTerm)),
        switchMap((searchTerm: string) => this.searchService.getSearchResults(searchTerm))
    )
    .subscribe((results: SearchCardData[]) => this.searchResults = results);
```

1. We kick things off by observing the keyup event on the search field as stated previously.
2. Then we `debounceTime` by 300ms so that we ignore keystrokes that happen within 300ms of each other. This signals that the user is in the middle of typing so don't send any emissions through to the next operator until they have at least paused their typing for at least 300ms.
3. Next is a `map` operator that pulls the value of the search input out of the event itself and passes just the value on to the next operator, `filter`.
4. The `filter` is not passing through any search values that are not at least 3 characters long. This is another superfluous request denial, assuming that the user doesn't want to search anything with just two characters.
5. Next is `distinctUntilChanged` which will not pass through any value until it is different from the previous value, preventing a duplicate request.
6. The `tap` operator is a useful one that will perform some side effect like update some value outside of the observable and ultimately pass through it's emission unchanged. In this case, if `tap` receives an emission of "dog", then it will push it to the searchTerms array and pass "dog" onto the `switchMap`.
7. It is the `switchMap` that ultimately calls the service, which is itself an observable. We use `switchMap` here because we want to cancel the current request if a different search value is received.
8. Then we subscribe to the result of the search service call, setting our array of results to the screen as data cards. We'll see those in a moment.

Next, let's take a look at the service call:

```ts
getSearchResults(searchTerm: string): Observable<SearchCardData[]> {
    const searchUrl = 'https://www.googleapis.com/customsearch/v1';
    const options = {
        params: new HttpParams()
            .set('key', this.apiKey)
            .set('cx', this.searchContext)
            .set('q', searchTerm)
    };

    return this.httpClient.get<SearchResult>(searchUrl, options)
        .pipe(
            pluck('items'),
            map(items => items.map(item => {
                return {
                    title: this.resolveTextLength(item.title, 35),
                    snippet: this.resolveTextLength(item.snippet, 125),
                    pageUrl: item.link,
                    image: {
                        url: this.resolveSearchResultImage(item.pagemap),
                        alt: item.title.split(' ')[0]
                    }
                };
            }))
        );
}
```

When we make the API call to Google for search results, we get a pretty complex object back. We want to reduce that data down to what the UI component cares about and ultimately wants to show.

1. Our source observable is the http request itself.
2. The returned data is wrapped in an object with a key of "items". The `pluck` operator will pluck off the key and give us just the value of "items" to pass on to the `map` operator.
3. The `map` operator receives the value of "items" and then pass through a transformed array as an emission to the subscription in the search field component. Inside the `map` operation, the items array is doing it's own JavaScript map over each of it's items, reducing the text so that the number of characters on the card for title and snippet are cut off at a certain length and an ellipsis is shown. The URL to the source of the result as well as an image (if there is one) is also set.

The result is something like this:

![Observables](/assets/img/stellar-search.gif)

## Subjects

Finally, let's take a quick peek at Subjects, which are like observers and observables at the same time.

![Say Whaaat!](/assets/img/say-whaaat.jpg)

Subjects are dynamic in that you can easily pass them new values from non-observable sources by utilizing a Subject's `next` method. For instance, we go back to the `tap` operation in our search field observable, we see that we are pushing each new search value into an array of searchTerms. What happens if we turn this into a Subject?

```ts
tap((searchTerm: string) => this.searchTerms.next(searchTerm))
```

Now we can subscribe to the searchTerms Subject and do something with that data reactively, like print the number of searches made during this session. This part isn't implemented in the Stellar Search app but might look something like this in the component's code:

```ts
this.seachTerms
    .subscribe(term => this.requestCount++);
```

## Alright, Ready to get Reactive!?

Just to summarize everything covered here:

* An observable emits data over time in an asynchronous fashion.
* Operators perform actions on that data and themselves return an observable.
* In order to ultimately consume the result of an observable, it must be subscribed to.
* Some observables complete themselves by design, while others must be unsubscribed from.
* RxJS is a reactive library for JavaScript/TypeScript.
* Reactive libraries exist for other languages too, including Rx.NET and RxJava.
* Angular is a front-end framework with reactive programming baked into it through RxJS.
* Remember to check out [ReactiveX](http://reactivex.io/) for all things reactive programming

Thanks for stopping by!
