---
title: "Monitor your Java Services with Prometheus Healthchecks"
excerpt: "Prometheus Healthchecks is a way to perform application health checks within the Prometheus platform."
permalink: /prometheus-healthchecks/
header:
  overlay_image: /assets/images/ecg.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: How do you feel today?
categories:
  - Programming
tags:
  - java
  - javaee
  - jakartaee
  - prometheus
---

{% include toc %}

# What is an Application Health Check?

> An Health Check measures the health status of an application.

A check can be seen as a _ping_ command: an **healthy** response means that the application connects to the service and that the service is operational, a **failing** check won't tell you what is wrong, but it can quickly point you in the right direction[^1]!

# The missing piece

[Prometheus](https://prometheus.io/docs/prometheus/latest/getting_started/) is a powerful monitoring system that gives you an insight of your application, and comes with different metric types.

Because of its simplicity (but also beauty) it doesn't come with an Health Check System, the only information available is the [automatically generated](https://prometheus.io/docs/concepts/jobs_instances/#automatically-generated-labels-and-time-series) time serie `up` which is a [gauge](https://prometheus.io/docs/concepts/metric_types/#gauge) that monitors a job's instance and can have two values:

* `1` if the instance is healthy, i.e. reachable
* or `0` if the scrape failed

## What if...

...we want to monitor the health status of a service from which our java application depends?

With the [Prometheus JVM Client](https://github.com/prometheus/client_java) we could build a _probe_ using a gauge and programmatically set a value of `1` or `0` depending on whether the system responds or not.
This solution come with a defect: it activates the "_probe_" action if and only if an event triggers the gauge metric collection.

But what if we could not only abstract this mechanism but also automate (schedule?!) the collection of health checks metrics?

## ...Custom Collector

This task requires a "Custom Collector".

Implementing the `io.prometheus.client.Collector` abstract class I've written the `HealthChecksCollector`, its duty is to keep a reference of every Health Check object and report the result to the `CollectorRegistry` using a gauge metric.

Every Health Check object must extends the `HealthCheck` abstract class, the `HealthStatus check()` method must perform the health check logic and returns the result using the `HealthStatus` enum (it can also throw an exception which will result in a failed health check).

You can find more details in the [API Document](https://strengthened.github.io/prometheus-healthchecks/apidocs/).

Here is a simple example:

```java
class DbHealthCheck extends HealthCheck {
  @Override
  public HealthStatus check() throws Exception {
    return checkDbConnection() ? HealthStatus.HEALTHY : HealthStatus.UNHEALTHY;
  }
  boolean checkDbConnection() {}
}

HealthChecksCollector healthchecksMetrics =
    HealthChecksCollector.newInstance().register();

DbHealthCheck dbHealthCheck = new DbHealthCheck();
FsHealthCheck fsHealthCheck = // ...

healthchecksMetrics.addHealthCheck("database", dbHealthCheck)
    .addHealthCheck("filesystem", fsHealthCheck);
```

Resulting in these samples:

```
# HELP health_check_status Health check status results
# TYPE health_check_status gauge
health_check_status{system="database",} 1.0
health_check_status{system="filesystem",} 0.0
```

# Where can I find it?

[Strengthened Projects](https://github.com/strengthened) is a GitHub organization where I'm going to put all my open-source software.

You can find more details about the project and other examples on [its site](https://strengthened.github.io/prometheus-healthchecks/).
The artifacts are available in the Central Maven Repository using the dependency:

```xml
<dependency>
  <groupId>com.github.strengthened</groupId>
  <artifactId>prometheus-healthchecks</artifactId>
  <version>LATEST</version>
</dependency>
```

# Let's keep in touch

What do you think about this project?

Do you have any question/suggestion/criticism/bug report? Feel free to submit an [issue](https://github.com/strengthened/prometheus-healthchecks/issues) or contact me through one of my social network accounts. I'll be happy to hear your feedback!

<blockquote class="twitter-tweet" data-lang="it"><p lang="en" dir="ltr">Monitor your <a href="https://twitter.com/java?ref_src=twsrc%5Etfw">@java</a> Services with <a href="https://twitter.com/PrometheusIO?ref_src=twsrc%5Etfw">@PrometheusIO</a> and Healthchecks! <a href="https://twitter.com/hashtag/JakartaEE?src=hash&amp;ref_src=twsrc%5Etfw">#JakartaEE</a> <a href="https://twitter.com/hashtag/Java?src=hash&amp;ref_src=twsrc%5Etfw">#Java</a> <a href="https://twitter.com/hashtag/JavaEE?src=hash&amp;ref_src=twsrc%5Etfw">#JavaEE</a> <a href="https://twitter.com/hashtag/PrometheusIO?src=hash&amp;ref_src=twsrc%5Etfw">#PrometheusIO</a><a href="https://t.co/DrnrJJV6FY">https://t.co/DrnrJJV6FY</a> <a href="https://t.co/p9ha74LpmX">pic.twitter.com/p9ha74LpmX</a></p>&mdash; Filippo (@filippomito) <a href="https://twitter.com/filippomito/status/975382292557455364?ref_src=twsrc%5Etfw">18 marzo 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[^1]: This is the reason health checks are recommended to be used for simple up/down checks and not for variable/metric-related checks.