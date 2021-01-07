---
layout: page
title: Memory model
---

The memory model put into action in the [C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf)
standard is the foundation of all the multithreading support available to us today.
Understanding it is crucial even if one does not wish to write low level
structures for special uses cases. At the very least it should be a humble lesson
to us all to have a cursory understanding on what complexity lies under the hood,
so we don't make carelessly very costly mistakes.

This page will collect all the necessary definitions in order of their use and then
explore what this means in action. The notes put forward here are not meant to
substitute a detailed description of the topic, but to rather serve as a quick
reference to those already familiar with the concepts. For detailed explanations
please refer to [C++ Concurrency in Action 2nd Edition - Anthony Williams](https://www.amazon.com/C-Concurrency-Action-Anthony-Williams/dp/1617294691/ref=sr_1_2?dchild=1&qid=1609941541&refinements=p_27%3AAnthony+Williams&s=books&sr=1-2&text=Anthony+Williams).

## Definitions

**[Object](https://en.cppreference.com/w/cpp/language/object)** - A C++ object has
size, alignment requirement, storage duration, lifetime, type, value and
optionally a name. It is a region of storage that can be manipulated by a C++
program. A variable of type **int** is an object just like compound objects as a
**struct** made up of multiple variables with type **int** is an object.

**[Memory location](https://en.cppreference.com/w/cpp/language/memory_model#Memory_location)** - A memory location is either an object of scalar
type or a maximal sequence of adjacent bit-fields all having nonzero width.
An object takes up one or more memory locations.

**[Undefined behavior](https://en.cppreference.com/w/cpp/language/ub)** - Behavior
for which the C++ standard does not impose any requirements. In other words,
anything can happen. The program can crash, enter into an invalid state etc.

**[Conflicting operation](https://isocpp.org/files/papers/N4860.pdf#subsubsection.6.9.2.1)** -
Two expression evaluations conflict if one of them modifies a
memory location and the other one reads or modifies the same memory
location simultaneously.

**[Data race](https://en.cppreference.com/w/cpp/language/memory_model#Threads_and_data_races)** -
When two or more threads can execute conflicting operations, it is called a data
race. In the presence of a data race the behavior of the program is undefined.

**Atomic operation** - An indivisible operation. The modification either fully
takes effect or not at all.

**[Sequenced before](https://isocpp.org/files/papers/N4860.pdf#subsection.6.9.1)** -
Sequenced before is an asymmetric, transitive, pair-wise
relation between evaluations executed by a single thread (6.9.2), which induces a
partial order among those evaluations. Given any two evaluations **A** and **B**
if **A** is sequenced before **B**, then the execution of **A** shall precede the
execution of **B**. If **A** is not sequenced before **B** and **B** is not
sequenced before **A**, then **A** and **B** are unsequenced.

**Synchronizes-with** - __Atomic__ write operation **A** is said to
synchronize-with __atomic__ read operation **B** if operations **A**, **B** and
every __atomic__ read-modify-write operation between **A** and **B** are suitably
tagged.

**Inter thread happens before** - Between threads operation **A** happens before
operation **B** if **A** synchronizes-with **B**.

**Happens before** - Operation **A** is said to happen before operation **B** if

- **A** is sequenced before **B**
- **A** inter-thread happens before **B**

Release sequence

There is a **Strongly happens before** relationship as well and it comes into play
only when using __memory_order_consume__. The use of __memory_order_consume__ is
discouraged by the standard since C++17 and for that reason we won't examine it.

## Memory orders
