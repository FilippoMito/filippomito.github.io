---
title: "Run your Eclipse IDE with Shenandoah GC"
excerpt: "Shenandoah is the low pause time garbage collector that reduces GC pause times by performing more garbage collection work concurrently with the running Java program"
permalink: /eclipse-shenandoah/
header:
  overlay_image: /assets/images/Shenandoah.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: Shenandoah National Park
categories:
  - Programming
tags:
  - java
toc: true
---

# At a glance

This post is a revisitation of my previous one on [ZGC](/eclipse-zgc/). Unlike ZGC, Shenandoah is also available for Windows and macOS!

---

[OpenJDK Wiki](https://wiki.openjdk.java.net/display/shenandoah/Main) states that Shenandoah does the bulk of GC work concurrently, including the concurrent compaction, which means its pause times are no longer directly proportional to the size of the heap. Garbage collecting a 200 GB heap or a 2 GB heap should have the similar low pause behavior.

## Availability

Shenandoah is in upstream OpenJDK since JDK 12, under JEP 189. Backports to JDK 8u and JDK 11u are available as well.

## Enabling Eclipse

My setup includes:

* OpenJDK 12 (LTS) from [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk12&jvmVariant=hotspot) macOS x64
* Eclipse Destop IDE from [Eclipse](https://www.eclipse.org/downloads/packages/) macOS x64

Please note that Shenandoah GC is available only on _HotSpot_ JVM, _OpenJ9_ JVM come with different GCs.

Download and install OpenJDK and Eclipse IDE, you can simply unpack the archives in you home directory, take note of installation paths.

### Eclipse startup

Locate the `eclipse.ini` configuration file in your Eclipse installation directory.

Add the `-vm` option following [these instructions](https://wiki.eclipse.org/Eclipse.ini#Specifying_the_JVM), you may end up with something like:

```text
-vm
/path/to/jdk-12.0.1/bin/java
```

Change the line containing `-XX:+UseG1GC` with:

```text
-XX:+UnlockExperimentalVMOptions
-XX:+UseShenandoahGC
```

You may also change the heap size parameters `Xms` and `Xmx` to higher values, personally I have quadrupled the values, more on this [here](https://wiki.openjdk.java.net/display/shenandoah/Main#Main-Basicconfiguration).

The result should look like:

```text
-vm
/opt/java/jdk-12.0.1/bin/java
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
-XX:+UseShenandoahGC
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