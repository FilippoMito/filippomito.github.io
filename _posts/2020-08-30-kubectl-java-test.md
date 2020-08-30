---
title: "I've developed a kubectl plugin in Java, and I'm proud of it!"
excerpt: "Here it is a simple kubectl plugin written in Java language and runned using jbang. Detailed guide later."
permalink: /kubectl-java-test/
header:
  overlay_image: /assets/images/k8s-java.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: Simple kubectl java plugin using jbang
categories:
  - Programming
  - DevOps
tags:
  - kubernetes
  - tools
  - java
  - jbang
toc: true
---

# Show me the code

Come up and have a look at it here:

[https://github.com/filippobuletto/kubectl-java-test](https://github.com/filippobuletto/kubectl-java-test)

**Please note that this isn't a true kubectl plugin, this is just sample code!**

java-test plugin gets pod names and other information, you can limit the namespace scope or use a regex pattern to search for pod names.

```bash
$ kubectl java-test -h
Usage: Simple kubectl plugin [-hV] [-n=NAMESPACE] [NAMEPATTERN]
Gets Pods names and other stuff
      [NAMEPATTERN]   Regex pattern
  -h, --help          Show this help message and exit.
  -n, --namespace=NAMESPACE
                      The namespace
  -V, --version       Print version information and exit.
```

## Install

Install [Java 11+](https://adoptopenjdk.net/installation.html) and [jbang](https://jbang.dev/download) and then:

```bash
# Note: the .java extension must be omitted in order to make kubectl able to recognize the plugin
# See https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/#naming-a-plugin
curl -Lo kubectl-java_test https://github.com/filippobuletto/kubectl-java-test/releases/latest/download/kubectl-java_test.java
sudo install -m755 kubectl-java_test /usr/local/bin
rm kubectl-java_test
```

## Usage

```
kubectl java-test -n <namespace> <pod_name_regex>
```

# But how can I do it?

More on how to do it yourself later!
