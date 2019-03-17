---
title: "Improve your stream collector-fu"
excerpt: "Have you ever find yourself wanting to collect a stream multiple times? This post is for you! :punch:"
permalink: /stream-collector-fu/
header:
  overlay_image: /assets/images/data-flow.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: The stream that flows in you also flows in the CPU.
categories:
  - Programming
tags:
  - java
toc: true
---

# The key

> The key is to collect / reduce

Stream in Java make possible functional-style operations on (collections of) elements, this makes the developer forget the good ol' `for` construct very quickly.

Sometimes the lazy developer, plagiarized by the stream _zen laziness_, focuses heavily on intermediate operations like `map` or `filter`, forgetting **the key**: collectors!

# Standard Collectors

The Java standard library provides many useful default collectors and simple API to build them, they can be found through the `java.util.stream.Collectors` class:

```java
// Compute sum of salaries of employee
int total = employees.stream()
        .collect(Collectors.summingInt(Employee::getSalary)));

// Group employees by department
Map<Department, List<Employee>> byDept
    = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

## Specialized Streams

> Sometimes forgotten things hide the best treasures

If you don't have lived under a rock you know that Java have some _specialized_ stream types: `IntStream`, `LongStream`, and `DoubleStream` that are streams over primitive `int`, `long` and `double` types.

Specialized streams can be used to obtain useful data from the stream, so if you need to calculate the maximum value of a stream of ints you can use the `OptionalInt max()` method of `IntStream`!
Someone can object that the same result can be obtained with a reduce operation like `reduce(Integer::max)` but what if you want to calculate an average?
You could to it using a custom and cumbersome collector, or go straight to the `OptionalDouble average()` method!

But wait, reading the `IntStream` JavaDoc, lurking around a corner, a precious gem appears!

> `summaryStatistics()`

This method returns an [IntSummaryStatistics](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/IntSummaryStatistics.html) that collects statistics such as count, min, max, sum, and average!

# Obtaining custom statistics from a Stream

Let's have a POJO representing a person and build a list of them:

```java
class Person {
    int weight;
    int height;
    int getHeight() {}
    int getWeight() {}
}

// Some random people
var list = List.of(new Person(67, 178),
                   new Person(86, 186),
                   new Person(73, 173));
```

In the good old days (pre Java 8) to find the group's maximum height and average weight we would have written something like this:

```java
int count = 0;
int maxHeight = Integer.MIN_VALUE;
int sumWeight = 0;
for (Person p : list) {
  count++;
  maxHeight = Math.max(maxHeight, p.getHeight());
  sumWeight += p.getWeight();
}
double avg = count > 0 ? (double) sumWeight / count : 0.0d;
```

But we live in a bright present and we have streams!

```java
var averageWeight = list.stream()
    .mapToInt(Person::getWeight).average();
var averageHeight = list.stream()
    .mapToInt(Person::getHeight).average();
averageWeight.ifPresent(aWeigh ->
    System.out.printf("Averages weight %.2f%n", aWeigh));
averageHeight.ifPresent(aHeigh ->
    System.out.printf("Averages height %.2f%n", aHeigh));

// Averages weight 75,33
// Averages height 179,00
```

This snipped is better (and more readable) but still we are **consuming two streams** to obtain two results! This is unacceptable.

A first improvement may be using the aforementioned `IntSummaryStatistics` to obtain a bunch of statistics from each person's measure.

```java
IntSummaryStatistics statsWeight =
    list.stream()
        .mapToInt(Person::getWeight)
        .summaryStatistics();
System.out.printf("Weight stats %s%n", statsWeight);
IntSummaryStatistics statsHeight =
    list.stream()
        .mapToInt(Person::getHeight)
        .summaryStatistics();
System.out.printf("Height stats %s%n", statsHeight);

// Weight stats IntSummaryStatistics{count=3, sum=226, min=67, average=75,333333, max=86}
// Height stats IntSummaryStatistics{count=3, sum=537, min=173, average=179,000000, max=186}
```

Simple and straight result, but still not very efficient: the stream is rebuilt and consumed twice.

## Learn from the JDK

Let's look at the `IntSummaryStatistics` JavaDoc:

```java
IntSummaryStatistics stats =
    intStream.collect(IntSummaryStatistics::new,
                      IntSummaryStatistics::accept,
                      IntSummaryStatistics::combine);
```

In the docs the author is building a **custom collector**, shall we do the same? Yes!

## Anatomy of a Collector

> A [Collector](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collector.html) is specified by four functions that work together to accumulate entries into a mutable result container, and optionally perform a final transform on the result.

They are:

- creation of a new result container: **supplier()**
- incorporating a new data element into a result container: **accumulator()**
- combining two result containers into one: **combiner()**
- performing an optional final transform on the container: **finisher()**

Let's build:

```java
class PersonStats {
  // Mutable content
  int count = 0;
  int maxHeight = Integer.MIN_VALUE;
  int sumWeight = 0; // Used to calculate average
  // Accumulator
  public void accept(Person p) {
    count++;
    maxHeight = Math.max(maxHeight, p.getHeight());
    sumWeight += p.getWeight();
  }
  // Combiner
  public PersonStats combine(PersonStats other) {
    count += other.count;
    maxHeight = Math.max(maxHeight, other.maxHeight);
    sumWeight += other.sumWeight;
    return this;
  }
  // Results
  public int getCount() {
    return count;
  }
  public int getMaxHeight() {
    return maxHeight;
  }
  public double getAvgWeight() {
    return getCount() > 0 ? (double) sumWeight / count : 0.0d;
  }
}
```

And now the final collector:

- the supplier is the `PersonStats` implicit constructor
- the accumulator is the `accept` method
- the combinator is (you guessed it) the `combine` method
- the optional finisher is absent because the `PersonStats` is the final result of the reduction, so we set the `IDENTITY_FINISH` characteristic

```java
static Collector<Person, ?, PersonStats> PERSON_STATS =
  Collector.of(PersonStats::new,
               PersonStats::accept,
               PersonStats::combine,
               Collector.Characteristics.IDENTITY_FINISH);
```

It's runtime! As simple as:

```java
PersonStats stats = list.parallelStream()
                        .collect(PERSON_STATS);
System.out.printf("Person stats %s%n", stats);

// Person stats {count=3, max height=186, avg weight=75,33}
```

Not only we have consumed the stream once, but we have even lifted the parallelism provided by the API!

### Adding the finisher, finally

Some smart people out there have already spotted a severe flow in the previous beauty: we are returning a `PersonStats` that is

- **mutable**
- an _intermediate container_

An immutable object is far more suitable in a functional style code, also the intermediate container can be hidden from the user eyes because contains only collector-specific logic!

Recapping with another example, here it is the intermediate container:

```java
static class PeopleFinder {
  private Person heavy;
  private Person tall;
  // Accumulator
  void accept(Person p) {
    heavy = heaviest(heavy, p);
    tall = tallest(tall, p);
  }
  // Logic
  Person heaviest(Person a, Person b) {
    return a == null ||
        (a.getWeight() < b.getWeight()) ? b : a;
  }
  Person tallest(Person a, Person b) {
    return a == null ||
        (a.getHeight() < b.getHeight()) ? b : a;
  }
  // Combiner
  PeopleFinder combine(PeopleFinder other) {
    heavy = heaviest(heavy, other.heavy);
    tall = tallest(tall, other.tall);
    return this;
  }
  // FInisher
  BigPeople result() {
    return new BigPeople(heavy, tall);
  }
}
```

The immutable result object:

```java
static class BigPeople {
  final Person heavy;
  final Person tall;
  BigPeople(Person heavy, Person tall) {}
  Optional<Person> heavyPerson() {}
  Optional<Person> tallPerson() {}
}
```

Finally, the new shiny collector:

```java
static Collector<Person, ?, BigPeople> PEOPLE_FINDER =
  Collector.of(PeopleFinder::new,
               PeopleFinder::accept,
               PeopleFinder::combine,
               PeopleFinder::result);
```

Have you noticed? We can even omit `PeopleFinder` intermediate container from the collector's type parameters!

```java
BigPeople bigPeople = list.parallelStream()
                          .collect(PEOPLE_FINDER);
System.out.printf("Big people %s%n", bigPeople);
// Big people {heaviest Optional[{weight=86, height=186}], tallest Optional[{weight=86, height=186}]}
```

# Final thoughts

What we learned from this post is that a Collector may seem like an obscure unknown black wizardry but that turned out to be simpler, and at the same time more powerful, than expected!

Happy collecting!