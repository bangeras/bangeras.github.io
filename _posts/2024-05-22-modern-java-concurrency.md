---
title: modern-java-concurrency
date: 2024-05-22 17:50:30 +0530
categories: [Java, Concurrency]
tags: [java, concurrency, virtualthreads]     # TAG names should always be lowercase
# mermaid: true
pin: true
---

[![](/assets/img/sourcecode.png "Source Code: bangeras/electron-media-compressor"){: width="30" height="35" }](https://github.com/bangeras/modern-concurrency){:target="_blank"}{:rel="noopener noreferrer"}

## Virtual Threads
Java Virtual Threads, introduced in Project Loom and included in recent versions of the Java platform, represent a significant advancement in Java's concurrency model. 
Virtual threads can handle a high number of concurrent tasks, significantly improving the scalability of applications that require numerous simultaneous connections or operations (e.g., web servers, microservices).
They reduce the overhead associated with context switching and memory usage compared to platform threads.

### How It Works
Continuation-Based Model: Virtual threads leverage continuations, which allow the suspension and resumption of tasks at certain points. This mechanism lets the JVM manage virtual threads efficiently.
Blocking Calls: Virtual threads can handle blocking I/O operations without occupying OS threads, thanks to the underlying continuation model. This approach means that blocking a virtual thread does not block an OS thread.

![Virtual Threads](/assets/img/modern-java-concurrency/virtual-threads1.png){: width="700" height="400" }
![Virtual + Platform Threads](/assets/img/modern-java-concurrency/virtual-threads2.png){: width="700" height="400" }

### Advantages
Virtual threads are created in JVM heap, with minimal creation/memory footprint. A single JVM can easily manage 100K+ Virtual threads. This improves the ability to achieve greater horizontal scalability with minimal 
VMs

## Structured Concurrency
Simplify concurrent programming by introducing an API for structured concurrency. Structured concurrency treats groups of related tasks running in different threads as a single unit of work, thereby streamlining error handling and cancellation, improving reliability, and enhancing observability. 
Just like structured programming helps us understand synchronous code, Structured concurrency would do likewise for concurrent programming
Promote a style of concurrent programming that can eliminate common risks arising from cancellation and shutdown, such as thread leaks and cancellation delays.

### StructuredTaskScope
Structured Concurrency using StructuredTaskScope is a superior implementation than using ExecutorService, as the StructuredTaskScope sends a cancellation request as needed and ensures all sub-tasks are terminated.

### ScopedValue
Improves over InheritableThreadLocal by allowing us to share the values between parent and child thread efficiently without copying.
ThreadLocal - Unconstrained Mutability - Unconstrained Scope = Scope Value
ScopeValue gets GC'ed when the scope of the ScopedValue exits.
![ScopedValue](/assets/img/modern-java-concurrency/scopedvalue.png){: width="700" height="400" }


Primary intention here is call the handleUser() with the secondary intention of binding the user ScopedValue with bob. This ScopedValue is accessible within handleUser() and its stack

## Enabling early-access Java preview features (in IntelliJ)
![build.gradle](/assets/img/modern-java-concurrency/build.gradle.png){: width="700" height="400" }
![Run Config](/assets/img/modern-java-concurrency/run-config.png){: width="700" height="400" }


