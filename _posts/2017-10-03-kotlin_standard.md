---
layout: post
title:  "Kotlin Basics: Standard Extension Functions"
date:   2017-02-08 15:00:00
description: What's the difference between the kotlin extension functions in Standard.kt? When to use let, apply, run, takeUnless or also?
comments: true
categories:
- kotlin
permalink: kotlin-standard-extensions
---

Kotlin's most powerful feature (compared to Java) are probably Extension Functions.  
They allow you to add new functionality to an existing class, without using inheritance or reflection.
You can define your own extension functions and Kotlin ships with a lot of pre-defined ones.  
There are e.g. a lot extensions for collections, but also those that extend every class in your program. That's right - _every single class_ is extended by the functions in `Standard.kt`.   
When I started with Kotlin, I found them quite confusing. With this post, I hope to un-confuse them a little bit.

Before we get to the details, some basics.

An Extension Function is a function that adds new functionality to an existing class. It's functionally equivalent to a static utility function in Java, but we can call it from the object itself.

In Java, you might have something like this:

```
class Util {
  public static final boolean isNumeric(String receiver) {
     return reveiver.matches("\\d+");
  }
}
...

String myString = ...;

if(Util.isNumeric(myString)) ...
```

By using an extension function, Kotlin allows us to call the `isNumeric` method directly on the receiving object:

```kotlin
fun String.isNumeric(): Boolean {
  return this.matches("\\d+".toRegex())
}
...

val myString = ...

if(myString.isNumeric()) ...
```

Pretty cool, right? You might agree that using extension functions increases the code's readability.

The example shows how an extension function is defined. We use the `fun` keyword, followed by the type we want to extend (in this case `String`). Then a `.` and the name of the function. That's it!  
The extension function will always receive an implicit parameter `this`, which is the object we called the extension function on (in this case, `myString`).

Let's take a look at Kotlin's standard extension functions.  


###### run

```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R = block()
```

This looks a bit messy in the beginning. But it's actually quite simple.
`run` is a generic extension function on _any_ type `T`, that executes another extension function on this type (extension functions as parameters are symbolized as `T.()`) and returns the result of that function.

Consider this example:

```kotlin
val generator = PasswordGenerator()
generator.seed = "someString"
generator.hash = {s -> someHash(s)}
generator.hashRepititions = 1000

val password: Password = generator.generate()
```

Someone didn't quite think through the design of this password generator class. Its constructor does nothing, but it needs a lot of initialization.  To use this class, I need to introduce a variable `generator`, set all necessary parameters and use `generate` to generate the actual password. This is cool if I want to generate multiple passwords with the same generator, but if I throwaway the generator variable anyway, I can save some typing by using `run`:

```kotlin
val password: Password = PasswordGenerator().run {
       seed = "someString"
       hash = {s -> someHash(s)}
       hashRepetitions = 1000

       generate()
   }
```

Lambdas in Kotlin implicitly return the result of the last line. That's why I can omit the temporary variable and store the password directly. Because an extension function is passed to `run` I can also access the password generator's properties like `seed` or `hash` directly.  
This way "the things that belong together" stay together.
It's one line more, I give you that. But it's still less code with less redundancy I had to write.  


###### apply

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```

This function looks almost like `run`, but there is a slight difference. `apply` will always return `this`, which is the receiver of the extension function.  
That means that we can use `apply` for builder-style initialization, even if the designer of the class didn't implement it with a builder in mind.  
If we stick to the password generator example:

```kotlin
val generator = PasswordGenerator().apply {
       seed = "someString"
       hash = {s -> someHash(s)}
       hashRepetitions = 1000
   }

val password = generator.generate()
```

This is particularly useful if you need an object with the same settings more than once.  
It can also be used to avoid `init{}` blocks during the initialization of a class:

```kotlin
class Message(message: String, signature: String) {
  val body = MessageBody().apply {
    text = message + "\n" + signature
  }
}
```


###### let

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)
```

`let` is particularly useful to use it instead of a `null` check:

```kotlin
val fruitBasket = ...

val result = apple?.let {
  fruitBasket.add(it)
}
```

The book (`it`) will only be added to the shelf if it's not `null`.   
Notice that `let` will return the result of the last line in the lambda (which is the result of `add`).  
Just like `run`, `let` also helps to keep the scope of your variables small. Instead of `this` you have to use `it` or a custom variable name to reference it:

```kotlin
val fruitBasket = ...

appleTree.pick()?.let {
  fruitBasket.add(it)
}
```


###### also

```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }
```

Kotlin 1.1 introduced this new extension function called `also`. It fills the gap between `let` and `apply`.  
Just like `apply`, it will always return its receiver. But instead of an extension function it takes a normal function as an argument.

```kotlin
class FruitBasket {
    private var weight = 0

    fun addFrom(appleTree: AppleTree) {
        val apple = appleTree.pick().also { apple ->
            this.weight += apple.weight
            add(apple)
        }
        ...
    }
    ...
    fun add(fruit: Fruit) = ...
}
```

I renamed the implicit `it` to an explicit `apple` this time.  Assuming the function `appleTree.pick()` returns an apple, the weight of the whole basket increases.  
Notice that both the apple and the basket have a `weight` property.  If I had used `apply`, it would not be possible<sup>*</sup> to access the basket's `weight`. Since `apply` takes an extension function, `this` would refer to the apple and not the basket.  With `also` this is possible.   


###### takeIf and takeUnless

```kotlin
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? = if (predicate(this)) this else null

public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? = if (!predicate(this)) this else null
```

The last two functions I want to mention have also been added to Kotlin with 1.1.

`takeIf` does exactly that - it takes the receiver if it satisfies a condition.  
`takeUnless` inverts the condition and takes `this` only if the condition is not satisfied.

For example:

```kotlin
val redApple = apple.takeIf { it.color != RED }
val otherApple = apple.takeUnless { it.color == RED }
```

Those two methods are the functional equivalent to the `filter` function to collections, but they operate on a single variable.  


###### Summary

Function  | Argument  |  Returns | Use when
--|---|---|
`run`  | `this` | any result | Re-scope / avoid temporary variables
`apply`  | `this` | `this` | Builder all the things!
`let`  | `it` | any result | Re-scope / null checks
`also`  | `it` | `this` | You need `apply` but don't want to shadow `this`
`takeIf`  |`it` | `it` or `null`  | shorthand for `if(predicate(it)) it else null`
`takeUnless`  | `it` | `it` or `null` | shorthand for `if(!predicate(it)) it else null`

---

<sup>*</sup> You still can by using `this@FruitBasket`, but do you want to do this?
