---
title: "Configuration for MicroProfile vs. Viper Configuration"
excerpt: "MicroProfile Config is a solid standard with some implementation available but has some deadly weaknesses that Viper :snake: overcomes"
permalink: /microprofile-config-viper/
header:
  overlay_image: /assets/images/viper-config.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: Injecting configurations
categories:
  - Programming
tags:
  - java
  - javaee
  - microprofile
  - viper
toc: true
---

# What is MicroProfile

_MicroProfile_ was created as a means to collaborate with vendors, individuals, and organizations like Java user groups in an open forum, to rapidly bring microservices to traditional Java EE developers. Now the project has been moved under the Eclipse Foundation and renamed "Eclipse MicroProfile".

MicroProfile collect some relevant Java EE technologies such as **CDI** and also extends it with some new capabilities, such as **Configuration**!

## Configuration for MicroProfile (1.2)

The project [_rationale_](https://github.com/eclipse/microprofile-config/blob/1.2/README.adoc):

> The majority of applications need to be configured based on a running environment. It must be possible to modify configuration data from outside an application so that the application itself does not need to be repackaged.

I think this has been done by the developers in a "artisan" way for too long, and this specification comes with great blaze from the developers.

### The good

The configuration comes from a `ConfigSource`, because there are many of them they get sorted according to their _ordinal_ value. There are some _default_ sources:

* `System.getProperties()` (ordinal=400)
* `System.getenv()` (ordinal=300)
* all `META-INF/microprofile-config.properties` files on the ClassPath.
(default ordinal=100, separately configurable via a config_ordinal property inside each file)

Plus you can define a custom `ConfigSource` and define an _ordinal_ value typically between 0 and 200 to overcome `.properties` files on the ClassPath.

If you need dynamic `ConfigSources` you can also register a `ConfigSourceProvider`, more on that can be found [here](https://github.com/eclipse/microprofile-config/blob/1.2/spec/src/main/asciidoc/configsources.asciidoc).

### The ugly

The core Microprofile-Config mechanism is purely String/String based. Built-in Converters may provide some basic target types such as Boolean, Integer, Long, ...

Custom converters must translate a single String property into the target type `T`, this is done also if `T` has a Constructor with a String parameter, or has a static `T valueOf(String)`/`T parse(CharSequence)` method, or is an Enum.

### The bad

In this post I want to consider only the Dependency Injection way of use: configured values are injected using the `@Inject` and the `@ConfigProperty` qualifier.

Let's dig into ConfigProperty annotation.

```java
@Qualifier
/* Omitted */
public @interface ConfigProperty {
    @Nonbinding
    String name() default "";
    @Nonbinding
    String defaultValue() default UNCONFIGURED_VALUE;
}
```

As you can see there are two annotation parameters:

 * **name**: _The key of the config property used to look up the configuration value._
 * **defaultValue**: _The default value if the configured property value does not exist._

This leads to almost two weak points:

 1. configuration keys gets scattered into the source code and the only way to collect them all is to chase annotation references;
 2. same fate applies to default values, and even worse same key may have different ones!

```java
@Inject
@ConfigProperty(name = "myprj.some.timeout", defaultValue = "100")
private Long timeout;
```

In this example I've injected the property with key `myprj.some.timeout` which in case of absence will have the runtime value of 100, the built-in converter for `Long` values will apply.

# Viper (0.2.0)

Without going into too much [detail of Viper implementation](https://github.com/civitz/viper/blob/master/README.md), I'll focus on the enumeration type that represents the keys of the properties:

```java
@viper.CdiConfiguration
public enum Property {
  SOME_INT,
  SOME_URL,
  DYNAMIC_TIMEOUT;
}
```

and the viper-generated code:

```java
@Qualifier
/* Omitted */
public @interface PropertyConfiguration {
  @Nonbinding
  Property value() default Property.SOME_INT;
}
```

so we can inject the property values as:

```java
@Inject
@PropertyConfiguration(Property.DYNAMIC_TIMEOUT)
private Long timeoutViper;
```

This is pretty similar to the `@ConfigProperty` qualifier, but as you have noticed the annotation lacks the default value!

Let's customize the enum with some string key and default values!

```java
@viper.CdiConfiguration
public enum Property {
  SOME_INT("myprj.some.int"),
  SOME_URL("myprj.some.url"),
  DYNAMIC_TIMEOUT("myprj.some.dynamic.timeout", "100");
  String key;
  String defaultValue;
  Property(String key, String defaultValue) {
    this.key = Objects.requireNonNull(key);
    this.defaultValue = defaultValue; // Nullable
  }
}
```

But wait, there's more! What about values validation?

```java
// "Converters" for primitive types
@viper.CdiConfiguration(producersForPrimitives = true)
public enum Property {
  SOME_INT("myprj.some.int", isInteger()),
  SOME_URL("myprj.some.url", isNotEmpty()),
  DYNAMIC_TIMEOUT("myprj.some.dynamic.timeout", isLong(), "100");

  String key;
  Predicate<String> validator;
  String defaultValue;

  /* Omitted other constructors */
  Property(String key, Predicate<String> validator, String defaultValue) {
    this.key = Objects.requireNonNull(key);
    this.validator = Objects.requireNonNull(validator);
    this.defaultValue = defaultValue; // Nullable
  }

  @viper.CdiConfiguration.ConfigValidator
  public Predicate<String> getValidator() {
    return validator;
  }
}
```

This pretty simple annotated enum makes the viper code generator to build a `PropertyConfigurationBean` which:
 * validates the properties using the supplied `Predicate<String>`,
 * produces all the primitive types that have the `valueOf(String)` method (such as `Boolean` and the `Number` sub-classes)
 * read values from a `viper.ConfigurationResolver<E extends Enum<E>>`

## ConfigurationResolver

```java
public interface ConfigurationResolver<E extends Enum<E>> {
  /** Returns a string value of the configuration key passed as a parameter. */
  String getConfigurationValue(E key);
  /* Omitted */
}
```

Any implementation of this interface's `String getConfigurationValue(E)` method will return the String value associated with the enum value that acts as a property key.

The simplest implementation could be like:

```java
class MyConfigurationResolver implements ConfigurationResolver<Property> {
  Map<String,String> props = readProps();
  @Override public String getConfigurationValue(Property key) {
    String result = props.get(getConfigurationKey(key));
    return result == null ? key.getDefaultValue() : result;
  }
}
```

### Implementing multiple sources in Viper

Because Viper supports a single `ConfigurationResolver` we could write multiple implementation of it but we are left to deal with an ordering problem. The ordinal value is missing, so we have to extend the resolver such as:

```java
public interface SortableConfigurationResolver extends ConfigurationResolver<Property> {
  int getOrdinal();
}
```

This concept is borrowed from `ConfigSource`'s' `int getOrdinal()` method.

Now let's consider some implementations:

```java
// Reads form System.getProperties() with ordinal 400
class SysConfigurationResolver implements SortableConfigurationResolver{}
// Reads form System.getenv() with ordinal 300
class EnvConfigurationResolver implements SortableConfigurationResolver{}
// Reads form Map<String,String> with ordinal 100
class PropsConfigurationResolver implements SortableConfigurationResolver{}
```

Achieving our goal centralizing into a single `ConfigurationResolver`:

```java
@ApplicationScoped
public class MyConfigurationResolver implements ConfigurationResolver<Property> {
  private final List<ConfigurationResolver<Property>> resolvers;

  public MyConfigurationResolver() {
    final List<SortableConfigurationResolver> resolvers = new ArrayList<>();
    resolvers.add(new EnvConfigurationResolver());
    resolvers.add(new SysConfigurationResolver());
    resolvers.add(new PropsConfigurationResolver());
    this.resolvers =
        resolvers.stream()
            .sorted(Comparator.comparing(SortableConfigurationResolver::getOrdinal)
                              .thenComparing(obj -> obj.getClass().getName()))
            .collect(Collectors.toList());
  }

  @Override
  public String getConfigurationValue(Property key) {
    return resolvers.stream()
                      .map(c -> c.getConfigurationValue(key))
                      .filter(Objects::nonNull)
                      .findFirst()
                        .orElse(key.getDefaultValue());
  }

  @Override
  public String getConfigurationKey(Property key) {
    return key.getKeyString();
  }

}
```

# Code examples

I wrote an example demo application comparing WildFly Swarm extension for microprofile-config and Viper.

You can [grab the code](https://github.com/filippobuletto/microprofile-config-demo) here:

```
 $ git clone https://github.com/filippobuletto/microprofile-config-demo.git
```

 * Commit [45a036d](https://github.com/filippobuletto/microprofile-config-demo/commit/45a036d43afa1773e5a8cfeb5920b7d8973d08b6): first implementation with a single Viper `ConfigurationResolver`.
 * Commit [3d12bdd](https://github.com/filippobuletto/microprofile-config-demo/commit/3d12bdd67e88871c5e2c3550c6d645f143d1a192): add Viper support for multiple sources.

# Conclusions

Although I am a little biased towards Viper's solution I have to applaud the Eclipse Microprofile effort to bring a good industry-grade standard where Java EE always had (and still has) a big void.

With Viper on stage we have even another choice now! Open source community benefits from the collective innovation.

As always, thanks for reading!

# References

 * MicroProfile Configuration Feature ([https://github.com/eclipse/microprofile-config](https://github.com/eclipse/microprofile-config)) _Apache License 2.0_
 * WildFly/Swarm extension for Eclipse MicroProfile Config ([https://github.com/wildfly-extras/wildfly-microprofile-config/](https://github.com/wildfly-extras/wildfly-microprofile-config/)) _Apache License 2.0_
 * A generator and a framework for injecting configurations via CDI (Viper) ([https://github.com/civitz/viper/](https://github.com/civitz/viper/)) _MIT License_

Kudos to my colleague [Roberto](https://civitz.github.io/) for writing Viper.

<blockquote class="twitter-tweet" data-lang="it"><p lang="fr" dir="ltr">Configuration for MicroProfile vs. Viper Configuration <a href="https://twitter.com/hashtag/java?src=hash&amp;ref_src=twsrc%5Etfw">#java</a> <a href="https://twitter.com/hashtag/javaee?src=hash&amp;ref_src=twsrc%5Etfw">#javaee</a> <a href="https://twitter.com/hashtag/microprofile?src=hash&amp;ref_src=twsrc%5Etfw">#microprofile</a> <a href="https://twitter.com/hashtag/microprofileio?src=hash&amp;ref_src=twsrc%5Etfw">#microprofileio</a> <a href="https://t.co/NQHE5lbGHv">https://t.co/NQHE5lbGHv</a> <a href="https://t.co/SJZjLvCCQJ">pic.twitter.com/SJZjLvCCQJ</a></p>&mdash; Filippo (@filippomito) <a href="https://twitter.com/filippomito/status/947791025174609920?ref_src=twsrc%5Etfw">1 gennaio 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>