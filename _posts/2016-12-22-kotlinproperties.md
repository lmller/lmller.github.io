---
layout: post
title:  "Kotlin Basics: Properties vs. Functions (vs. Properties)"
date:   2016-12-22 15:00:00
description: Kotlin offers different methods to access an object's attribute. Properties, functions or computed properties; which should you choose?
comments: true
categories:
- kotlin
permalink: kotlin-properties-vs-functions
---
If you just started with Kotlin, there's a good chance that you are overwhelmed with the new possibilities.
This is especially true if you originally came from Java, where concepts like [Properties][930cbe90] or [Function Types][fc2a71b7] simply do not exist.  
In this post, I will address some of the questions concerning _Properties_ a beginner might have when starting with Kotlin.

###### Properties and Fields
Unlike Java, Kotlin *has no fields*.
Consider the two examples:

```java
public class Banana {
  public String color = "green"
}
```

```kotlin
class Banana {
  var color = "green"
}
```

On the first glance, they seem equivalent. But they aren't. Kotlin has no fields and `color` can therefore not be accessed directly but will be hidden behind `getColor` and `setColor` methods (so called accessor).   
These methods are generated for us by the Kotlin compiler. But we can write the `getColor` and `setColor` methods ourselves, if we want to.

```kotlin
class Banana {
    var color = "green"
        get() = field
        set(value) {field=value}
}
```

Writing it like this is equivalent to the previous example, where we left out the `get()` and `set()` methods. The `field` identifier references the _backing field_ of this property. These backing fields are used to store the actual value of a property and are automatically generated if we use the default implementation of `get()` _or_ `set()` or if we write a custom accessor and use the `field` identifier.

Writing the accessors ourselves is pointless unless we want to do intercept the access of the property.  
Maybe we only want to allow certain values:

```kotlin
class Banana {
    var color = "green"
        set(value) {
          if(value == "yellow" || value == "green"){
            field = value
          }
        }
}
```

With this, the property (that is, the backing field) will not change unless the new value is either `"yellow"` or `"green"`.

###### "Computed" Properties vs. "Normal" Properties

As mentioned above, a backing field will not be generated unless we actually use it (be it directly by referencing `field` or indirectly by using a default accessor).  For properties that don't have a backing field, I will borrow the term _computed property_ from [Swift][f22c8840], since it's basically the same concept.

Let's change the example a bit so that it's using a computed property.

```kotlin
class Banana {
    var ripeness = 1

    var color: String = "green"
        get() = when {
            ripeness > 80 -> "brown"
            ripeness > 50 -> "yellow"
            else -> "green"
        }
}
```

I added a new mutable property `ripeness`. The old `color` property will now return a different result, depending on the value stored in `ripeness`. Because `color` is declared as `var`, and therefore a default `set` is generated, it will _still_ generate a backing field.
This backing field is really useless though:

```kotlin
val banana = Banana()
banana.color = "blue"

println(banana.color)

```

The `println(banana.color)` will never print `"blue"`. It will call the `get`-function and there fore return `"green"`.

Since we don't want useless backing fields (memory!) we get rid of the setter by declaring our property as `val`.

```kotlin
class Banana {
    var ripeness = 1

    val color: String
        get() = when {
            ripeness > 80 -> "brown"
            ripeness > 50 -> "yellow"
            else -> "green"
        }
}
```

This `val` property will not offer a `set` method.  

###### "Computed" Properties vs. Functions

Writing a computed property like this is essentially the same as writing a function.

```kotlin
class Banana {
    var ripeness = 1

    fun getColor: String = when {
            ripeness > 80 -> "brown"
            ripeness > 50 -> "yellow"
            else -> "green"
    }
}
```

But I prefer the property syntax. A rule of thumb can be _"if it describes the object, it's a property. If does something with the object or with another object, it's a function"._

I think the [C# explanation on "Choosing Between Properties and Methods"][00a58f3c] is useful for Kotlin as well.

  [00a58f3c]: https://msdn.microsoft.com/en-us/library/ms229054(v=vs.100).aspx "C#: "Choosing Between Properties and Methods""

###### A Possible Pitfall

With many new possibilities it easy to run into problems. One problem I had was forgetting the `get` for a property.

```kotlin
class Banana {
    var ripeness = 1

    val color: String = when {
            ripeness > 80 -> "brown"
            ripeness > 50 -> "yellow"
            else -> "green"
        }
}
```

While this is syntactically correct, it is not what we actually want.
By omitting the `get` accessor, we _assign the value of the backing field_. This assignment takes place during the construction of our `Banana` object. This means that the result will be computed once and then never again. **Since the `ripeness` is initially set to `1`, calling `banana.color` will always return `"green"`.**

###### Learn more

The [Properties page][930cbe90] is a good starting point, as well as [Visibility Modifiers][ccdf2b54], as they can also be applied to properties.  
Some more advanced property topics include [Overriding Properties][27e1d06e] and [Delegated Properties][ac96a7e8]

  [fc2a71b7]: https://kotlinlang.org/docs/reference/lambdas.html#function-types "Kotin: Function Types"
  [930cbe90]: https://kotlinlang.org/docs/reference/properties.html "Kotlin: Properties"
  [ac96a7e8]: https://kotlinlang.org/docs/reference/delegated-properties.html "Kotlin: Delegated Properties"
  [27e1d06e]: https://kotlinlang.org/docs/reference/properties.html#overriding-properties "Kotlin: Override Properties"
  [ccdf2b54]: https://kotlinlang.org/docs/reference/visibility-modifiers.html "Kotlin: Visibility Modifiers"

  [f22c8840]: https://developer.apple.com/swift/ "Swift"
