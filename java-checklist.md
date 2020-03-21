# Java Overview

Contents:
- [JVM memory](#jvm-memory)
- [JVM Garbage Collection](#jvm-gc)
    - [Reference Counting](#jvm-gc-rc)
    - [Tracing](#jvm-gc-tracing)

<a name="jvm-memory"></a>
## JVM memory

JVM memory is logically split on next spaces:
- **Heap** (GC is working here) - assumption that most of objects are short-to-live and should be removed soon after allocation;
    - New generation - newly allocated objects, GC starts frequently;
    - Old generation - long-live objects, GC starts rarely.
- **Permanent Generation**:
    - classes metadata;
    - constant pool;
    - code methods / constructors;
    - static variables.

A little more detailed logical structure of the Heap:
----------------
|Eden|S1|S2|Old|
----------------
**Eden** is a space for newly created objects;
S1, S2 are **survivors**, spaces for objects that alived after some GC;
**Old** is a space for long-lived objects.

Two *survivor* spaces used to avoid fragmentation. Process of GC there:
- GC process works in S1;
- All marked objects are copied to S2 without fragmentation;
- Clear S1;
- GC process works in S2;
- ...

<a name="jvm-gc"><a/>
## JVM Garbage Collection

There are 2 common techniques to implement GC process:
1. Reference Counting
2. Tracing

[#](#jvm-gc-rc) A **reference counting** technique provides a counter of references for every object. When the counter drops to 0 it means that there is no references to the object, so it is unreachable now and could be safely recycled. Objects are released immediately as the counter dropped. Handling the counter in runtime leads to performance degradation but in other hand there is no STW (stop-the-world) pauses.

The first problem in this technique is a **circular references**. It's a situation when two objects are referenced each other. An example is a parent object P that references to a child object C (P --> C) and back (C --> P). This situation protects them from being released. The common approach to fix this is to use references with different **strength** (strong, weak). The rule is that object couldn't be released if there is at least one *strong* reference to the object. But if all references to an object are *weak* so the object could be recycled. In the example the fix is to make back reference weak (C --weak--> P). In that case it's now possible to destroy parent object while there is a weak reference to it.

The second problem is handling object references that shared in multiple threads. It leads to additional complexity (atomicity is a requirement) while implementing the counter.

[#](#jvm-gc-tracing) A **tracing** technique provides a number of root points for garbage collection (GC). These points are:
- Class --> static variables;
- Thread --> thread locals;
- Local stack --> local variables and method parameters;
- JNI Local and Global;
- Monitors and Locks;
- Hold by JVM --> System classloader, execption handling.

Every cycle of GC starts tracing beginning from the root points and mark all objects that could be accessed from the root and deeper. Every object that isn't marked in the process is considered obsolete and safely removed. 

