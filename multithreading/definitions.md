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
sequenced before **A**, then **A** and **B** are unsequenced. An expression **X**
is said to be sequenced before an expression **Y** if every value computation
and every side effect associated with the expression **X** is sequenced before
every value computation and every side effect associated with the expression **Y**.

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

## Modification order

From the standard [6.9.2.1.4](https://isocpp.org/files/papers/N4860.pdf#subsection.6.9.2):
"All modifications to a particular atomic object **M** occur in some particular
total order, called the modification order of **M**. [Note: There is a separate
order for each atomic object. There is no requirement that these can be combined
into a single total order for all objects. In general this will be impossible
since different threads may observe modifications to different objects in
inconsistent orders.— end note"

That is a dense paragraph. First of all it only specifies a total modification
order for atomic objects only! Non atomic objects have no total modification order
in a multithreaded environment. Secondly, it does not specify a total global order
across all atomic object modifications. This can be enforced by only using the
default memory ordering for all atomic operations: [memory_order_seq_cst](memory_orders.md#sequentially-consistent-ordering).

What is a total modification order for an atomic object and what does it mean to
have one across all atomic objects?

To be able to formulate what any of this means one has to remember how a
modern computer works. For our inspection it is enough to focus on the memory and
the CPU.

![Image for cpu, memory and caches]()

All programs consist of a set of instructions and operate on a set of data. Both
the instruction set and the data lives on the hard drive (nowadays SSD) at first.
When used, for quicker access, they get loaded into the working memory the RAM. To
execute the instructions and to manipulate the data, the necessary information is
loaded into the cache lines of the CPU. The CPU does it's thing and writes the
results back into the cache line, which then at some point will be copied back into
the RAM. That is rather simple if you have only one core with it's own cache. But
nowadays we have many and some caches are shared some are not.

### No total modification order

What does no total modification order really mean? A simple example where two
threads modify the same non atomic variable demonstrates just that.
Both threads are only incrementing the same value by one in a loop. As a final
value we would like to see a 100, but that is not quite what we will get.

Code:
```c++
code example here
```

Outputs:
```bash
result here
```

What we see is that the result will be unspecified, because we are executing
conflicting operations, causing a data race, which is undefined behavior.

Following the CPU cache lines, train of thought something like below is happening:

![Image for the example]()

### Total modification order

Using the exact same example as before, but substituting the non atomic object for
an atomic one we will see that the result will always be a 100. There is a total
modification order for the atomic variable on which both threads have agreed upon.
This changes from execution to execution, but the results after each modification
are well defined and consequently the final result is as well.

Code:
```c++
code example here
```

Outputs:
```bash
result here
```

### Total modification order across all atomic operations

This can only happen if all atomic operations are using [memory_order_seq_cst](memory_orders.md#sequentially-consistent-ordering) ordering. If there is any that
are not, than there will be no total modification order.

## Sequenced-before, Happens-before and Synchronizes-with

### Sequenced-before

[Sequenced-before](https://en.cppreference.com/w/cpp/language/eval_order) is a
complicated topic with a great number of nuances. It defines
how expressions, sub-expressions and operands get evaluated in what order.
For example:

```c++
int x;  // A
x = 10; // B
++x;    // C
```

Expression **A** is sequenced before **B** and that is sequenced before **C**. On
the other hand it, the evaluation of functions arguments for example is not
sequenced. For example:

```c++
#include <cstdio>

int print_a()
{
    return std::puts("a");
}

int print_b()
{
    return std::puts("b");
}

int main()
{
    return print_a() + print_b();
}
```

The output on stdout could be "a b" or "b a". It is not specified which argument of
the + operator is evaluated first. In this case __print_a__ and __print_b__ are
unsequenced.

For our purposes it is not necessary to know the exactly how the sequencing
rules work, but it is important to have a tangible example for grasping the
concept and knowing that details exist that we have conveniently ignored.

With sequenced-before cleared up we can quickly understand what happens-before and
synchronizes-with mean.

### Happens-before and synchronizes-with

In a single threaded sense happens-before is the same as sequenced-before. If
operation **A** is sequenced before operation **B** then operation **A** also
happens-before operation **B**. Simple enough. The multithreaded interaction is
just as simple but another condition must be satisfied for the inter-thread
happens-before to take effect. That condition is the synchronizes-with. By having
a synchronizes-with interaction between two atomic operations, being executed in
two threads, is like having a bridge connecting the two. This could be
conceptualized as if there were no threads at all, but the set of instructions
took effect as if they happened in some sequence after each other like in the
simple single threaded example.

![Image to show the difference between multithreaded and single]()

Whenever happens-before is mentioned it both means inter-thread happens-before and
single thread happens-before (sequenced-before).

## Release sequence
