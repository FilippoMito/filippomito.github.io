---
title: "How to trash the exception model in Java"
excerpt: "Getting inspired by Scala, a functional perspective :underage:"
permalink: /trash-the-exception-model/
header:
  overlay_image: /assets/images/miracle.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: _You should be more explicit in step two._
categories:
  - Programming
tags:
  - java
  - vavr
  - scala
toc: true
---

# Error Handling With Try and Either

When programming in Java you can no longer run away from handling _errors_ and _exceptions_ in your code. Throwing and catching exceptions is a "glorified goto" and also hidden performance costs (why is creating a `Throwable` so expensive? It's because usally it will call `Throwable.fillInStackTrace()`).

Let’s first have a look at this approach:

```java
public class Customer {
    private final int age;
    public int getAge() {
        return age;
    }
}
public class Alcoholic {
    private final int alcoholConcentration;
    public int getAlcoholConcentration() {
        return alcoholConcentration;
    }
}
public class UnderageException extends Exception {
    public UnderageException(String message) {
        super(message);
    }
}
Alcoholic buyAlcoholic(Customer customer) throws UnderageException {
    if (customer.getAge() < 18) {
        throw new UnderageException("Age: " + customer.getAge());
    }
    return new Alcoholic(5);
}

try {
    Customer tooYoung = new Customer(16);
    Alcoholic alcoholic = buyAlcoholic(tooYoung);
} catch (UnderageException ex) {
    // Troubles
}
```

Nasty, isn't it? And doesn’t really go well with functional programming (or other situation like concurrency).

## The functional way

In **Scala**, there is a specific type that represents computations that may result in an exception: `Try` (there is also a similar type, called `Either`, but we'll cover it later), is there a way to get it in **Java** also?

[VAVR](http://www.vavr.io/vavr-docs/#_try) to the rescue!

> Try is a monadic container type which represents a computation that may either result in an exception, or return a successfully computed value. Instances of `Try`, are either an instance of `Success` or `Failure`.

An instance of `Success<A>` simply wraps a value of type `A`, an instance of `Failure<A>` wraps a _Throwable_ (i.e. an exception or other kind of error).

```java
Try<Alcoholic> alcoholic = Try.of(() -> buyAlcoholic(new Customer(16)));
```

This beauty returns a value of type `Try<Alcoholic>`. If the given Customer has a legal age, this will be a `Success<Alcoholic>`, otherwise the `buyAlcoholic` method throws a _UnderageException_, however, it will be a `Failure<Alcoholic>`.

Working with Try instances is actually very similar to working with Option values: you can check if a Try is a success by calling `isSuccess` and then retrieve the wrapped value with `get`, but there aren’t many situations where you will want to do that.

Normally you want to return a default value (using `orElse*` or `getOrElse*` methods) or transform a failure into a success (using `recover*` or `recoverWith*` methods).

### Chain operations

Like Option, Try supports all the higher-order methods you know from other monadic containers:

```java
Try<Integer> concentration = alcoholic.map(Alcoholic::getAlcoholConcentration);
```

Mapping a `Try<Alcoholic>` that is a `Success<Alcoholic>` to a `Try<Integer>` results in a `Success<Integer>`. If it’s a `Failure<Alcoholic>`, the resulting `Try<Integer>` will be a `Failure<Integer>`, on the other hand, containing the same exception as the `Failure<Alcoholic>`.

### Filter and foreach

It is also passible to _filter_ a Try or call _foreach_ on it.

The filter method returns a `Failure` if the Try on which it is called is already a `Failure` or if the predicate passed to it returns false (in which case the wrapped exception is a `NoSuchElementException`).
The function passed to foreach is executed only if the Try is a `Success`, which allows you to execute a _side-effect_.

### Pattern Matching

If you want to know whether a `Try` instance you have received as the result of some computation represents a success or not and execute different code branches depending on the result you make use of pattern matching:

```java
String result = Match(concentration).of(
        Case($Success($()), c -> "drinked " + c),
        Case($Failure($()), t -> "troubles"));
```

### Recovering

If recover is called on a `Success` instance, that instance is returned as is. Otherwise, if the partial function is defined for the given `Failure` instance, its result is returned as a `Success`.

```java
Try<Integer> recovered = concentration
    .recover(th -> Match(th).of(
        Case($(instanceOf(UnderageException.class)), () -> 0),
        Case($(), () -> -1)));
```

## Try conclusion

The `Try` type allows to encapsulate computations that result in errors in a container and to chain operations on the computed values in a very elegant way.

In the next part we are going to deal with `Either`, an alternative type for representing computations that may result in errors, but with a wider scope of application that goes beyond error handling.

<blockquote class="twitter-tweet" data-lang="it"><p lang="en" dir="ltr">How to trash the exception model in Java (Part 1)<a href="https://t.co/d1hFOXnTDW">https://t.co/d1hFOXnTDW</a></p>&mdash; Filippo (@filippomito) <a href="https://twitter.com/filippomito/status/942533003112341504?ref_src=twsrc%5Etfw">17 dicembre 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>