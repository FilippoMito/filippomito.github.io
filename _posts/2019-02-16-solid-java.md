---
title: "SOLID Java"
excerpt: "SOLID Principles in Java, address dependencies and tight coupling. Create a set of well designed and written classes so you can speed up the coding process! :rocket:"
permalink: /solid-java/
header:
  overlay_image: /assets/images/solid.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: Standing on Solid Rock is good
categories:
  - Programming
tags:
  - java
toc: true
---

# SOLID Java

SOLID states for five design principles that help a developer build easy to extend and maintain software:

**S** – Single-responsibility principle

**O** – Open-closed principle

**L** – Liskov substitution principle

**I** – Interface segregation principle

**D** – Dependency Inversion Principle

In this post I’m not going to explain what’s hidden behind it though, it’s been already done in dozens of articles all over the web!

This is intended to **Java developers** who already know SOLID and just would like to have a quick-and-dirty way to remember about them and have a quick insight into these rules in case they forgot what any of them was exactly about.

# Single Responsibility Principle

> A class should have one, and only one, reason to change.

A class should have only **one responsibility** which means class should be highly cohesive and implement strongly related logic. Class implementing feature 1 AND feature 2 AND feature 3 (and so on) violates SRP.

## SRP Example

```java
// BAD
public class UserSettingService {
  public void changeEmail(User user) {
    if (checkAccess(user)) {
       //Grant option to change
    }
  }
  public boolean checkAccess(User user) {
    //Verify if the user is valid.
  }
}

// GOOD
public class UserSettingService {
  public void changeEmail(User user) {
    if (securityService.checkAccess(user)) {
       //Grant option to change
    }
  }
}
public class SecurityService {
  public static boolean checkAccess(User user) {
    //check the access.
  }
}
```

## SRP Code smells

- more than one contextually separated piece of code within single class
- large setup in tests (TDD is very useful when it comes to detecting SRP violation)

## SRP Goodies

- separated classes responsible for given use case can be now reused in other parts of an application
- separated classes responsible for a given use case can be now tested separately

# Open/closed Principle

> You should be able to extend a classes behavior, without modifying it.

Class should be open for extension and closed for modification. You should be able to extend class behavior without the need to modify its implementation (how? Don’t modify existing code of class X but write a new piece of code that will be used by class X).

## OCP Example

```java
// BAD
public class Logger {
  String logging;
  public Logger(String logging) {
    this.logging = logging;
  }
  public void log() {
    if ("console".equals(logging)) {
      // Log to console
    } else if ("file".equals(logging)) {
      // Log to file
    }
  }
}

// GOOD
public interface Log {
    void log();
}
public class ConsoleLog implements Log {
  void log() {
    // Log to console
  }
}
public class FileLog implements Log {
  void log() {
    // Log to file
  }
}
public class Logger {
  Log log;
  public Logger(Log log) {
    this.log = log;
  }
  public void log() {
    this.log.log();
  }
}
```

## OCP Code smells

- if you notice class X directly references other class Y from within its code base, it’s a sign that class Y should be passed to class X (either through constructor/single method) e.g. through dependency injection
- complex if-else or switch statements

## OCP Goodies

- class X functionality can be easily extended with new functionality encapsulated in a separate class without the need to change class X implementation (it’s not aware of introduced changes)
- code is loosely coupled
- injected class Y can be easily mocked in tests

# Liskov Substitution Principle

> Derived classes must be substitutable for their base classes.

This is an extension of open/close principle. Derived classes **should not change the behavior of the base class** (behavior of inherited methods). Provided a class Y is a subclass of class X any instance referencing class X should be able to reference class Y as well (derived types must be complete substitutes for their base types).

## LSP Example

```java
// BAD
public class DataHashSet extends HashSet {
  int addCount = 0;
  public boolean function add(Object object) {
    addCount++;
    return super.add(object);
  }
  // the size of count will be added twice!
  public boolean function addAll(Collection collection) {
    addCount += collection.size();
    return super.addAll(collection);
  }
}

// GOOD: Delegation Over Subtyping
public class DataHashSet implements Set {
  int addCount = 0;
  Set set;
  public DataHashSet(Set set) {
    this.set = set;
  }
  public boolean add(Object object) {
    addCount++;
    return this.set.add(object);
  }
  public boolean addAll(Collection collection) {
    addCount += collection.size();
    return this.set.addAll(collection);
  }
}
```

## LSP Code smells

- if it looks like a duck, quacks like a duck but needs batteries for that purpose - it’s probably a violation of LSP
- modification of inherited behavior in subclass
- exceptions raised in overridden inherited methods

## LSP Goodies

- avoiding unexpected and incorrect results
- clear distinction between shared inherited interface and extended functionality

# Interface Segregation Principle

> Make fine grained interfaces that are client specific.

Once an interface is becoming too large / fat, we absolutely need to split it into small interfaces that are more specific. And interface will be defined by the client that will use it, which means that client of the interface will only know about the methods that are related to them.

## ISP Example

```java
// BAD
public interface Car {
  Status open();
  Speed drive(Gas gas);
  Engine changeEngine(Engine newEngine);
}
public class Driver {
  public Driver(Car car) {}
  public Speed ride() {
    this.car.open();
    return this.car.drive(new Gas(10));
  }
}
public class Mechanic {
  public Mechanic(Car car) {}
  public Engine fixEngine(Engine newEngine) {
    return this.car.changeEngine(newEngine);
  }
}

// GOOD
public interface RidableCar {
  Status open();
  Speed drive(Gas gas);
}
public interface FixableCar {
    Engine changeEngine(Engine newEngine);
  }
public class Driver {
  // Same with RidableCar
}
public class Mechanic {
  // Same with FixableCar
}
```

## ISP Code smells

- one fat interface implemented by many classes where none of these classes implement 100% of interface’s methods. Such fat interface should be split into smaller interfaces suitable for client needs

## ISP Goodies

- highly cohesive code
- avoiding coupling between all classes using a single fat interface (once a method in the single fat interface gets updated, all classes - no matter they use this method or not - are forced to update accordingly)
- clear separation of business logic by grouping responsibilities into separate interfaces

# Dependency Inversion Principle

> Depend on abstractions, not on concretions.

If your implementation detail will depend on the higher-level abstractions, it will help you to get a system that is coupled correctly. Also, it will influence the encapsulation and cohesion of that system.

## DIP Example

```java
// BAD
public class SQLDatabase {
  public void connect() {
    String connectionstring = System.getProperty("MSSQLConnection");
    // Make DB Connection
  }
  public Object search(String key) {
    // Do SQL Statement
    return query.find();
  }
}
public class DocumentDatabase {
  // Same logic but with document details
}

// GOOD
public interface Connector {
  Connection open();
}
public interface Finder {
  Object find(String key);
}
public class MySqlConnector implements Connector {}
public class DocumentConnector implements Connector {}
public class MySqlFinder implements Finder {}
public class DocumentFinder implements Finder {}

public class Database {
  public Database(Connector connector,
                  Finder finder) {
    this.connector = connector;
    this.finder = finder;
  }
  public Connection connect() {
    return connector.open();
  }
  public Object search(String key) {
    return finder.find(key);
  }
}
```

## DIP Code smells

- instantiation of low-level modules in high-level ones
- calls to class methods of low-level modules/classes

## DIP Goodies

- increase reusability of higher-level modules by making them independent of lower-level modules
- dependency injection[^1] can be employed to facilitate the run-time provisioning of the chosen low-level component implementation to the high-level component
- injected class can be easily mocked in tests

# Update

Removed CDI code from Dependency Inversion example because it isn't necessary.

[^1]: [@Inject for the Java Platform.](http://weld.cdi-spec.org/)