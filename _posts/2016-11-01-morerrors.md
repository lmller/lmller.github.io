---
layout: post
title:  Increase the Java compiler's error-limit in Gradle.
date:   2016-11-01 15:00:00
description: Quick guide on how to increase the number of displayed errors in the gradle console.
comments: false
categories:
- android
- gradle
- java
permalink: more-errors
---
When updating a library or doing a big refactoring it quickly happens that we run against a massive wall of compiler errors.
The Java compiler only displays around 100, so you never quite know how much progress you already made.

###### Is it possible to increase the 100-Errors limit?

Luckily, `javac` has a handy flag `-Xmaxerrs` which we can use for that.
But we build our Apps using Gradle and the Gradle Android plugin, so we do not have direct control of the executed commands.

The Android Gradle plugin uses the default `JavaCompile` task that is shipped with Gradle. We can therefore regain control of the compiler options.

An easy way of applying flags like `-Xmaxerrs` is to use Gradle's `afterEvaluate` [lifecycle callback](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:project_evaluation):

> You can receive a notification immediately before and after a project is evaluated. This can be used to do things like performing additional configuration once all the definitions in a build script have been applied, or for some custom logging or profiling.

Sounds perfect.  
Now, the task we need to configure is `JavaCompile`, and we want to set the compiler arguments of this task.  
`CompileOptions.compilerArgs` is a list of Strings. It's basically the representation of everything behind the `javac` command, where every space starts a new list element. So when I want to add an element `-Xmaxerrs 2000` to the list, I need to add an element `-Xmaxerrs` **and another element** `2000` directly in a row.  
The list looks like this:`["-Xmaxerrs", "2000"]`

In groovy, we can use the left-shift (`<<`) operator to add two elements in a single line.

The result is:

```
afterEvaluate {
    tasks.withType(JavaCompile) {
        options.compilerArgs << "-Xmaxerrs" << "2000"
    }
}
```

If you add this snippet to your app's `build.gradle`, the compiler will not stop on 100 errors anymore. You can finally see the complete mess you produced.
