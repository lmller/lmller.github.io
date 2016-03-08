---
layout: post
title:  "A note on TestSubscriber"
date:   2016-03-08 09:00:00
description: Something I discoverd while TDD-ing a RxJava App
categories:
- tests
permalink: testsubscriber
---
####I
When using the [RxJava TestSubscriber](http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html) to test Observables, I have to make sure that I don't use `subscribeOn` or `observeOn`.
```
val testSubscriber = TestSubscriber<ThingsIWant>()
DataSource().subscribeOn(Schedulers.test())
            .subscribe(testSubscriber)

testSubscriber.assertValueCount(42)
```
Will not work (although the `Schedulers.test()`) 

Also, for the `TestSubscribers` to have more useful methods, I should always use Rx version `1.1.0` and above.

####II
I shouldn't forget that (in Kotlin) the `subscibe` method that takes a lambda 
````
DataSource().subscribe {
    //do lambda stuff
}
````
is only for lambdas - and not for instances of Subscriber. 
This seems obvious, but it happens that while changing things from one concept to the other I might accidently keep the '{}' brakets.

####III
Good recources on Testing Rx:
- http://fedepaol.github.io/blog/2015/09/13/testing-rxjava-observables-subscriptions/
- http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html
