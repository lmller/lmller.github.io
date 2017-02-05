---
layout: post
title:  "Notes on Rx Tests"
date:   2016-03-08 09:00:00
description: Something I discovered while TDD-ing a RxJava App
categories:
- tests
- rx
permalink: testsubscriber
---
While trying to test-drive a RxJava app in Kotlin, I ran into some problems. 
Here are my notes on that topic.

#### 1

When using the [RxJava TestSubscriber](http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html) to test Observables, I have to make sure that I don't use `subscribeOn` or `observeOn` (RxJava version `1.1.0` introduced many methods that make the TestSubscriber actually pretty useful).

```kotlin
val testSubscriber = TestSubscriber<ThingsIWant>()
DataSource().subscribeOn(Schedulers.test())
            .subscribe(testSubscriber)

testSubscriber.assertValueCount(42)
```

Will not work even though the `Schedulers.test()` looks like it's made for... tests.
The [documentation](http://reactivex.io/RxJava/javadoc/rx/schedulers/Schedulers.html#test()) is not clear on that it simply says: 

> Creates and returns a TestScheduler, which is useful for debugging. 
> It allows you to test schedules of events by manually advancing the clock at whatever pace you choose.

So the `TestScheduler` doesn't mean "blocking" or "on the same thread" as the rest, it just means I could manually advance the clock.

Good guy [David Karnok](https://twitter.com/akarnokd) helped me understanding the TestScheduler on twitter:

<blockquote class="twitter-tweet" data-conversation="none" data-lang="de"><p lang="en" dir="ltr"><a href="https://twitter.com/lovisbrot">@lovisbrot</a> Use ts.awaitTerminalEvent before asserting; store test() and call advanceTimeBy to make time move forward.</p>&mdash; David Karnok (@akarnokd) <a href="https://twitter.com/akarnokd/status/707233438672281600">8. März 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

#### 2

I shouldn't forget that (in Kotlin) the `subscribe` method that takes a lambda 

```kotlin
DataSource().subscribe {
    //do things
}
````

is only for lambdas - and not for instances of Subscriber. 
This seems obvious, but it happens that while changing things from one concept to the other I might accidently keep the `{}` brackets.


#### 3

Good recources on Testing Rx:

- <http://fedepaol.github.io/blog/2015/09/13/testing-rxjava-observables-subscriptions/>
- <http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html>
- <http://reactivex.io/RxJava/javadoc/rx/schedulers/TestScheduler.html>
