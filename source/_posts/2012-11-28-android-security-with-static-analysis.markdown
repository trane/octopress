---
layout: post
title: "Static Analysis of Android: Introduction"
date: 2012-12-01 17:00
comments: true
categories: [static analysis, security]
tags: [static analysis, cesk, dalvik, android, security]
---

#Introduction
Malware on Android [is a problem][]. With over [500 million][] activated
devices, it is a large user base to attack and the Android Market is a great
delivery mechanism. Even though Google has started to scan applications uploaded
to the marketplace, there still has been an [up-tick in malware][].
This raises the central question I am addressing in my Undergraduate Thesis:
Can an Android application be proven to be non-malicious by small step analysis?

This article starts off a series of articles that I will be writing over the
course of the next few months investigating this question through my research
with the [U-Combinator][] group at the [University of Utah][]. The group is
determining, through static analysis, how much can be proven about the nature of
any Android application in an effort to provide a relatively-provable secure
Android environment. To get started, this article gives an overview of what
makes up an Android application.

<!-- more -->

#Android and Dalvik
Android applications are really just Dalvik byte-code that run on the
[Dalvik virtual machine][]. They start out as collections of .java files and get
translated into a single [Dalvik Executable file][] (`dex`). Every application,
for security reasons, has its own independent address space and memory.

#Dalvik VM and Instruction Set
The goals of the Dalvik VM are to run on a device with relatively little RAM and
a slow CPU. Since each application will run in its own address space and memory,
special care was taken to shave off as much memory taken up by each compiled
Dalvik program.

The VM is a register based VM, like Parrot - the VM used for [Rakudo Perl 6][].
A register design was chosen over the stack based design of Java because of the
need to significantly reduce the CPU operation overhead associated with pushing
and popping duplicate operands. This decision, however, became less important
after a [JIT was introduced][] in Android. Most of the gains from the register
design come from reducing instructions and interpreter-killing opcode
dispatches.

Registers can be 16, 256, or 64k bits
depending on the instruction. Unlike Parrot's infinite register machine, Dalvik
has a finite number of registers, though it is an incredibly large number of
registers - 2^16=65,536 [to be exact][].

The [instruction set][] is small compared to a behemoth like x86. The opcodes
fit within 32-bits and, from a higher-level abstraction, can be represented by a
little more than 40 instructions.

#Dalvik Executable File - dex

Each dex file starts with a *File Header*, which provides meta data about
itself, such as file size, checksums.
The last two parts of a dex file are the Class definitions and the program
data. In the middle are a handful of tables.

##Application Global Lookup Tables
In order to shave off some memory overhead: strings, types, method prototypes,
fields,
and methods are all put into separate tables and referenced in the executable
code by indexing into these tables. For example, the string table holds every
string in the file, including string constants, class names, method names, and
variable names to name a few.

##Reference Graphs
Each of these sections can and do reference other sections. The *data section*
will directly use each of the lookup tables to execute its logic, while the
*class definitions* might reference and index into the *methods table* which
references an index to the *prototype table* which then references the *type
table* which references the *string table* which can then go back and reference
the *data section*.

##Shared Pool
While that seems convoluted and complicated, the result is that dex files end up
with a *shared pool* of data among all of the java classes, thus reducing the
memory footprint when they are translated into dex. This sharing continues with
built-in libraries, which upon starting an Android device, get loaded into
memory immediately.

This immediately raises into question the idea that every application has its
own memory and address space. It is safer to say that private memory stays
private, like application heap and dex structures or application-specific dex
files. For shared memory, data that can be cleanly ejected by the kernel at any
time or copy-on-write heap data is accessible from Zygote.

##Zygote
Zygote is a Dalvik process that starts on boot that will ensure that classes are
loaded before they are needed. It helps to give process-sandboxed
applications access to shared libraries and read-only copy-on-write heap data.
Zygote is forked at will and provides access to the
Zygote parent, ensuring that data that *can* be shared is shared, and data that
*can't* be shared is kept private.

##Garbage Collection
Not surprisingly, with the constraints given by Dalvik for sharing data, garbage
collection must help to keep private data away from other memory. The GC uses a
Mark and Sweep approach. In many cases, reachability bits don't
need to be separated from the main heap. However, with Android, these mark bits
are kept separate from other heap memory.

Garbage collection is also not a global process, there are separate GCs along
with separate processes and heaps. To further complicate it, the GCs must be
able to respect the shared pool.

#Verification and Optimizations
Some work is done up-front during an application's install to ensure that the
dex file structure is sound. This is includes checking for valid indices and
offsets into lookup tables, checking types and references. This, however, is not
a security measure. This only ensures that the code is valid. Perfectly "honest"
dex can behave maliciously and as I'll write about in future articles can
circumvent the built-in platform security.

If possible, lookup table operations are removed by
static linking, such as turning a method name string lookup, it can index into
a v-table.

#Conclusion
Most of the design decisions of the Dalvik VM have little to do with the
abstractions made by Static Analysis. However, it is important to understand the
inner workings of a system before you can make the assumptions required to infer
maliciousness.

Knowing, for example, that Garbage collection does not need to factor into
security concerns when analysing an application is helpful when asserting the
behavior of an application.

Ultimately, the analysis of an Android application is not limited in the ways
that the application is. But, knowing the constraints of an application aids in
creating a realistic model to analyse.

[Dalvik virtual machine]: http://en.wikipedia.org/wiki/Dalvik_(software)
[is a problem]: https://www.google.com/search?q=malware+android
[500 million]: http://news.cnet.com/8301-1035_3-57510994-94/google-500-million-android-devices-activated/
[University of Utah]: http://www.utah.edu
[U-Combinator]: http://www.ucombinator.org/
[up-tick in malware]: http://techcrunch.com/2012/11/05/android-malware-surges-despite-googles-efforts-to-bounce-dodgy-apps-off-its-platform-f-secure-ids-51447-unique-samples-in-q3/
[instruction set]: http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html
[Dalvik Executable file]: http://www.retrodev.com/android/dexformat.html
[Rakudo Perl 6]: http://rakudo.org
[JIT was introduced]: https://www.youtube.com/watch?v=Ls0tM-c4Vfo
[to be exact]: http://codingrelic.geekhold.com/2010/07/virtual-instruction-sets-opcode.html
