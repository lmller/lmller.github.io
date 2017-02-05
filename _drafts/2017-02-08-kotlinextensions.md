---
layout: post
title:  "Kotlin Basics: Standard Extension Functions"
date:   2017-02-08 15:00:00
description: What's the difference between the kotlin extension functions in Standard.kt? When to use let, apply, run, when or also?
comments: true
categories:
- kotlin
permalink: kotlin-standard-extensions
---
Kotlin's most powerful feature (compared to Java) are probably Extension Functions.
They allow you to add new functionality to an existing class, without using inheritance or reflection.
You can define your own extension functions and Kotlin ships with a lot of pre-defined ones. There are e.g. a lot extensions for collections, but also those that extend every class in your program. That's right - _every single class_ is extended by the functions in `Standard.kt`. When I started with Kotlin, I found them quite confusing. With this post, I hope to un-confuse them a little bit.
