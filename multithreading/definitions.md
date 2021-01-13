---
layout: page
title: Definitions
---

## Memory model

The memory model put into action in the [C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf)
standard is the foundation of all the multithreading support available to us today.
Understanding it is crucial even if one does not wish to write low level
structures for special use cases. At the very least it should be a humble lesson
to us all to have a cursory understanding on what complexity lies under the hood,
so we don't make carelessly very costly mistakes.

This page will collect all the necessary definitions in order of their use and then
explore what they mean in action. The notes put forward here are not meant to
substitute a detailed description of the topics, but to rather serve as a quick
reference to those already familiar with the concepts. For detailed explanations
please refer to [C++ Concurrency in Action 2nd Edition - Anthony Williams](https://www.amazon.com/C-Concurrency-Action-Anthony-Williams/dp/1617294691/ref=sr_1_2?dchild=1&qid=1609941541&refinements=p_27%3AAnthony+Williams&s=books&sr=1-2&text=Anthony+Williams)
, the newest version of the standard [C++20](https://isocpp.org/files/papers/N4860.pdf)
or [cppreference.com](https://en.cppreference.com/w/).

## Definitions

In this section the most necessary definitions required to understand or reason
about multithreaded code have been collected and put at the top of the page to
be used as a quick reference. Each concept that is not immediately
apparent from the definitions alone, but is a result of the interplay of a multiple
of them is elaborated upon in the following sections.

> Some definitions have been partially butchered or they have been modified and
merged with other definitions to remove unnecessary complexity. These
discrepancies can be observed when accessing the linked sources. One of these
missing snippets of information is __memory_order_consume__. It's use is
discouraged by the standard in consumer code and multithreading is difficult enough
without it. By omitting it fully from the below discussion a number of complexities
were removed like **Strongly happens before** or **Dependency-ordered before**.
If this would be of need for you, please refer to the linked definitions to get
the fuller picture.
That said the below definitions are detailed enough that they can be relied upon
when reasoning about multithreaded code.

**[Object](https://en.cppreference.com/w/cpp/language/object)** - A C++ object has
size, alignment requirement, storage duration, lifetime, type, value and
optionally a name. It is a region of storage that can be manipulated by a C++
program. A variable of type **int** is an object just like compound objects as a
**struct** made up of multiple variables with type **int** is an object.
An object takes up one or more memory locations.

**[Memory location](https://en.cppreference.com/w/cpp/language/memory_model#Memory_location)** - A memory location is either an object of scalar
type (arithmetic type, pointer, enumeration or std::nullptr) or a maximal sequence
of adjacent bit-fields all having nonzero width. A memory location is not
necessarily accessible by the program, but is rather managed by the implementation.

**[Undefined behavior](https://en.cppreference.com/w/cpp/language/ub)** - Behavior
for which the C++ standard does not impose any requirements on. In other words,
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

**Synchronizes-with** - __Atomic write__ operation **A** is said to
synchronize-with __atomic read__ operation **B** if operations **A**, **B** and
every __atomic read-modify-write__ operation between **A** and **B** are suitably
tagged. (More on suitable tagging at the Memory ordering section.)

**[Inter thread happens before](https://en.cppreference.com/w/cpp/atomic/memory_order#Inter-thread_happens-before)** - Between threads operation **A**
happens before operation **B** if **A** synchronizes-with **B**.

**[Happens before](https://en.cppreference.com/w/cpp/atomic/memory_order#Happens-before)** -
Operation **A** is said to happen before operation **B** if

- **A** is sequenced before **B** or
- **A** inter-thread happens before **B**

**[Release sequence](https://isocpp.org/files/papers/N4860.pdf#subsection.6.9.1)** - A release sequence headed by a release operation **A** on an atomic
object __M__ is a maximal contiguous sub-sequence of side effects in the
modification order of __M__, where the first operation is **A**, and every
subsequent operation is an atomic read-modify-write operation.

To put everything together it is vital to understand what memory orderings are
available and what do they mean. Each atomic operation according to it's type can
be tagged with a memory ordering type.

## Synchronizes with

## Release sequence

## Modification order

From the standard [6.9.2.1.4](https://isocpp.org/files/papers/N4860.pdf#subsection.6.9.2):

"All modifications to a particular atomic object **M** occur in some particular
total order, called the modification order of **M**. [Note: There is a separate
order for each atomic object. There is no requirement that these can be combined
into a single total order for all objects. In general this will be impossible
since different threads may observe modifications to different objects in
inconsistent orders.— end note"

That is a dense paragraph. To be able to formulate an idea how this can even happen
one has to remember how a modern computer works. For our inspection it is enough to
focus on the memory and the CPU.



All programs consist on a set of instructions and operate on a set of data. Both
the instruction set and the data lives on the hard drive (nowadays SSD) at first.
When used, for quicker access, they get loaded into the working memory the RAM. To
execute the instructions and to manipulate the data, the necessary information is
loaded into the cache lines of the CPU. The CPU does it's thing and writes the
results back into the cache line, which then at some point will be copied back into
the RAM. That is rather simple if you have only one core with it's own cache. But
nowadays we have many and some caches are shared some are not. Considering having
two cores with their separate cache lines working on the same data **M** without
properly synchronizing on it. **M** has the value 3 for example. Thread A writes the value 3 into **M** while Thread B reads
it and
