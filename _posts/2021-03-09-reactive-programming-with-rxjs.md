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

[Imperative vs Declarative Approach](/assets/img/imperative-vs-declarative.png)

In both approaches, we initialize an array of 12 "raw" donuts straight off the assembly line. With the imperative approach, our code documents that we are going to iterate over each donut, determine if it is broken, add sprinkles if it is not broken and then add it to the box of sprinkledDonuts. With the declarative approach, we obfuscate the iteration of the for-loop by _implying_ it with our use of the chained filter and map methods. We can look at our declarative statement and read it as "box up the raw donuts that aren't broken and add sprinkles on them".

These two examples are written in Javascript, but you can see how the declarative approach resembles a pipeline with the filter and map methods. Keep this in mind as we head into authoring reactive programming.

## The Trinity of Reactive Programming

* Observables: receiving from a data source
* Operators: pipeline of actions on the data
* Subscriptions: consuming the final result

### Observables

Observables are the heart of the reactive programming paradigm in that we cannot have reactive programming without reading from a data source of some type. We can think of the donut assembly line in our previous example as an observable. Another example of an observable would be a social media feed that is constantly receiving new posts for display on the screen.

[Observables](/assets/img/observables.png)

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

works much like a reducer in that it will accumulate donuts until it gets to 12 (`index % 12`) and then initialize a new box with a single donut in it. We are using a bit of digital smoke and mirrors here because scan will emit a box with one donut, then a box with two donuts, _10 passes later_... then a box with twelve donuts, then a box with one donut... and so on. To emulate that our subscription only gets a box of 12 donuts, we utilize the filter operator, which will discard any box that has less than 12 donuts (Blasphemy, I know!):

```
filter(donuts => donuts.length === 12)
```


