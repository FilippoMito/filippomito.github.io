---
title: "Run your Eclipse IDE with ZGC"
excerpt: "The Z Garbage Collector, also known as ZGC, is a scalable low latency garbage collector that can improve responsiveness of you Java based IDE"
permalink: /eclipse-zgc/
header:
  overlay_image: /assets/images/zgc.png
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: ZGC is an experimental GC
categories:
  - Programming
tags:
  - java
toc: true
---

# At a glance

[OpenJDK Wiki](https://wiki.openjdk.java.net/display/zgc/Main) states that ZGC is:

    Concurrent
    Region-based
    Compacting
    NUMA-aware
    Using colored pointers
    Using load barriers

At its core, ZGC is a concurrent garbage collector, meaning all heavy lifting work is done **while Java threads continue to execute**. This greatly limits the impact garbage collection will have on your application's response time.

## Availability

ZGC is currently only available on Linux/x64, if you with to test it in other platforms you might use a VM or Docker, also JDK 11, JDK 12 or JDK 13 Early Access are needed.

## Enabling Eclipse

My setup includes:

* OpenJDK 11 (LTS) from [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk11&jvmVariant=hotspot) Linux/x64
* Eclipse Destop IDE from [Eclipse](https://www.eclipse.org/downloads/packages/) Linux/x64

Please note that ZGC is available only on _HotSpot_ JVM, _OpenJ9_ JVM come with different GCs.

Download and install OpenJDK and Eclipse IDE, you can simply unpack the archives in you home directory, take note of installation paths.

### Eclipse startup

Locate the `eclipse.ini` configuration file in your Eclipse installation directory.

Add the `-vm` option following [these instructions](https://wiki.eclipse.org/Eclipse.ini#Specifying_the_JVM), you may end up with something like:

```text
-vm
/path/to/jdk-11.0.3/bin/java
```

Change the line containing `-XX:+UseG1GC` with:

```text
-XX:+UnlockExperimentalVMOptions
-XX:+UseZGC
```

You may also change the heap size parameters `Xms` and `Xmx` to higher values, personally I have quadrupled the values, more on this [here](https://wiki.openjdk.java.net/display/zgc/Main#Main-SettingHeapSize).

The result should look like:

```text
-vm
/opt/java/jdk-11.0.3/bin/java
-startup
plugins/org.eclipse.equinox.launcher_1.5.300.v20190213-1655.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.gtk.linux.x86_64_1.1.1000.v20190125-2016
-product
org.eclipse.epp.package.jee.product
-showsplash
org.eclipse.epp.package.common
--launcher.defaultAction
openFile
--launcher.defaultAction
openFile
--launcher.appendVmargs
-vmargs
-Dosgi.requiredJavaVersion=1.8
-Dosgi.instance.area.default=@user.home/eclipse-workspace
-XX:+UnlockExperimentalVMOptions
-XX:+UseZGC
-XX:+UseStringDeduplication
--add-modules=ALL-SYSTEM
-Dosgi.requiredJavaVersion=1.8
-Dosgi.dataAreaRequiresExplicitInit=true
-Xms1024m
-Xmx4096m
--add-modules=ALL-SYSTEM
```

# Enjoy

Now you can run Eclipse IDE and enjoy it with your low-pause GC equipped JVM!