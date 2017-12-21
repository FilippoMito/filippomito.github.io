---
title: "How to trash the exception model in Java: the Either type"
excerpt: "Getting inspired by Scala, a functional perspective, part two :on:"
permalink: /trash-the-exception-model-either/
header:
  overlay_image: /assets/images/left-or-right.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: _Left or right?_
categories:
  - Programming
tags:
  - java
  - vavr
  - scala
---

{% include toc %}

# The Either Type

In the previous article, we discussed [functional error handling using Try](/trash-the-exception-model/), I also mentioned the existence of another similar type called `Either`.

From [VAVR docs](http://www.vavr.io/vavr-docs/#_either):

> `Either` represents a value of two possible types. An `Either` is either a `Left` or a `Right`.

_What on the green earth is that?_

Like `Try`, `Either` is a container type. Unlike the aforementioned one, it takes not only one, but two type parameters:
an `Either<A, B>` instance can contain _either_ an instance of `A`, or an instance of `B`. This is different from a Tuple, which contains `both` an `A` and a `B` instance.

By convention, a _Left_ value signifies a failure case result and a _Right_ value signifies a success. You can think of a (say) `Try<String>` as an `Either<Throwable, String>`.

## How to create an Either

```java
// Remember this?
Alcoholic buyAlcoholic(Customer customer) throws UnderageException {
    if (customer.getAge() < 18) {
        throw new UnderageException("Age: " + customer.getAge());
    }
    return new Alcoholic(5);
}
// From boring to fun!
public class UnderageError {
    private final String reason;
    public String getReason() {
        return reason;
    }
}
Either<UnderageError, Alcoholic> buyAlcoholicEither(Customer customer) {
    if (customer.getAge() < 18) {
        return Either.left(new UnderageError("Age: " + customer.getAge()));
    }
    return Either.right(new Alcoholic(5));
}
```

If we call `buyAlcoholicEither(new Customer(16))` we will get an `UnderageError` wrapped in a **Left**. If we pass in `new Customer(20)`, the return value will be a **Right** containing a `Alcoholic`.

## How to use an Either

You can know if an Either `isLeft` or `isRight`. You can also do pattern matching on it (which is the best way of working with objects of this type).

```java
Either<UnderageError, Alcoholic> alcoholicEither = //...
Match(alcoholicEither).of(
        Case($Right($()), c -> "drinked " + c),
        Case($Left($()), t -> "troubles")
        );
```

Once you have an `Either`, you can call `map` on it:

`Either<UnderageError,Integer> concentrationOrError = alcoholicEither.map(Alcoholic::getAlcoholConcentration);`

If it’s called on a `Right`, the value inside it will be transformed. If it’s a `Left`, that will be returned unchanged.

We can also change the _Right success_ convention using projections:

> If the given Either is a Right and projected to a Left, the Left operations have no effect on the Right value. If the given Either is a Left and projected to a Right, the Right operations have no effect on the Left value. If a Left is projected to a Left or a Right is projected to a Right, the operations have an effect.

`Either<String, Alcoholic> concentrationOrError = alcoholicEither.left().map(UnderageError::getReason).toEither();`

## Other Features

We can _fold_ Left and Right to one common type:

`String reasonOrAlcoholic = alcoholicEither.fold(UnderageError::getReason, Alcoholic::toString);`

or swap sides:

`Either<Alcoholic, UnderageError> swap = alcoholicEither.swap();`

`Either` supports all the higher-order methods: `map` ( and `mapLeft`), `flatMap`, `filter`, `forEach`, ...

### Either as an Exception replacement

One of the _nice_ features of Java exceptions is the ability to declare several different potential exception types as part of a method signature. Either can do that as well! And without the exception goto style handling and performance penalties:

```java
public interface Error {
    String getReason();
}
public class UnderageError implements Error {
    private final String reason;
    @Override public String getReason() {
        return reason;
    }
}
public class ImpossibleError implements Error {
    @Override public String getReason() {
        return "Black magic happened";
    }
}
Either<Error, Alcoholic> alcoholicEither = //...
Match(alcoholicEither).of(
        Case($Right($()), c -> "drinked " + c),
        Case($Left($(instanceOf(UnderageError.class))), t -> "troubles"),
        Case($Left($(instanceOf(ImpossibleError.class))), t -> "wizard")
        );
```

Nice and smooth as silk :smile:

# Conclusions and thoughts

When you learn a new paradigm, you need to reconsider all the familiar ways of solving problems. Functional programming uses different idioms to report error conditions!
`Either` is a type that is not without flaws, and whether you want to have to deal with them and incorporate it in your own code is ultimately up to you.

Memento: throwing an exception will break your functional composition and probably result in unexpected behaviour for the caller of your _function_. So it should be reserved as a method of last resort, for when the other options don’t make sense.

<blockquote class="twitter-tweet" data-lang="it"><p lang="en" dir="ltr">How to trash the exception model in Java: the Either type <a href="https://t.co/xVifwygChP">https://t.co/xVifwygChP</a></p>&mdash; Filippo (@filippomito) <a href="https://twitter.com/filippomito/status/944100359718531073?ref_src=twsrc%5Etfw">22 dicembre 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>