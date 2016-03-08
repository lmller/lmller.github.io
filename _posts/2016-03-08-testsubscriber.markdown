---
layout: post
title:  "Notes on TestSubscriber"
date:   2016-03-08 09:00:00
description: Something I discoverd while TDD-ing a RxJava App
categories:
- tests
- rx
permalink: testsubscriber
---
While trying to test-drive a RxJava App in kotlin, I ran into some problems. 
Here are my notes on that topic.

#### I

When using the [RxJava TestSubscriber](http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html) to test Observables, I have to make sure that I don't use `subscribeOn` or `observeOn` (RxJava version `1.1.0` introduced many methods that make the TestSubscriber actually pretty useful).

```kotlin
val testSubscriber = TestSubscriber<ThingsIWant>()
DataSource().subscribeOn(Schedulers.test())
            .subscribe(testSubscriber)

testSubscriber.assertValueCount(42)
```

Will not work even though the `Schedulers.test()` looks like its made for... tests.
The [documentation](http://reactivex.io/RxJava/javadoc/rx/schedulers/Schedulers.html#test()) is not clear on that it simply says: 

> Creates and returns a TestScheduler, which is useful for debugging. 
> It allows you to test schedules of events by manually advancing the clock at whatever pace you choose.

And for the [TestScheduler](http://reactivex.io/RxJava/javadoc/rx/schedulers/TestScheduler.html)

> The TestScheduler is useful for debugging. It allows you to test schedules of events by manually advancing the clock at whatever pace you choose.

So the `TestScheduler` doesn't not mean "blocking" or "on the same thread" as the rest, it just means I could manually advance the clock.

#### II

I shouldn't forget that (in Kotlin) the `subscibe` method that takes a lambda 

````
DataSource().subscribe {
    //do lambda stuff
}
````

is only for lambdas - and not for instances of Subscriber. 
This seems obvious, but it happens that while changing things from one concept to the other I might accidently keep the '{}' brakets.

#### III

Good recources on Testing Rx:

- http://fedepaol.github.io/blog/2015/09/13/testing-rxjava-observables-subscriptions/
- http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html
