---
layout: page
title: Memory orders
---

## Memory orders

Memory orderings specify how the memory is accessed for an atomic operation for
the object the operation is affecting and for objects around the atomic
operation. The effects of this mind bending sentence with the above definitions is
what makes multithreading work in C++. To understand why, first let's take a look
on the memory ordering tags, the building blocks of higher level interactions.

The memory ordering tags are:

- memory_order_seq_cst
- memory_order_relaxed
- memory_order_acquire
- memory_order_release
- memory_order_acq_rel

These can be used to "suitably tag" appropriate types of atomic operations. The
below table shows which type of operation can receive which type of tag.

|Type|Tags|
|-----|----|
|store| memory_order_seq_cst, memory_order_release, memory_order_relaxed |
|load| memory_order_seq_cst, memory_order_acquire, memory_order_relaxed |
|read-modify-write| memory_order_seq_cst, memory_order_acquire, memory_order_release, memory_order_acq_rel, memory_order_relaxed |

To which atomic operation belongs to which type, please refer to
[cppreference.com](https://en.cppreference.com/w/).

### [Sequentially consistent ordering](https://en.cppreference.com/w/cpp/atomic/memory_order#Sequentially-consistent_ordering)

Memory_order_seq_cst is the default ordering for any atomic operation for a good
reason. It is easiest to reason about. Simply put if the multithreaded application
relies exclusively on memory_order_seq_cst then it is said, that the application
behaves in a sequentially consistent way. In other words as if all operations of
the program were performed in a specified order, by a single thread. This
modification order is singular and global. All threads agree at all times on the
exact value of each atomic variable. No confusions, no headaches, but there is a
cost to this.

Each time, two atomic operations marked with memory_order_seq_cst synchronize-with
each other the processor has to execute costly operations synchronizing the
variables affected by the relevant happens-before relationships. On top of this
sequential consistency is only guaranteed if all operations are tagged with
memory_order_seq_cst on a given modification order. When this stops being the case
a previously sequentially consistent atomic operation can be thought of a
memory_order_release (on a store) or memory_order_acquire (on a load).

### [Relaxed ordering](https://en.cppreference.com/w/cpp/atomic/memory_order#Relaxed_ordering)

### [Release-Acquire ordering](https://en.cppreference.com/w/cpp/atomic/memory_order#Release-Acquire_ordering)

The above definitions are not easy to comprehend and given their interconnected
nature either the whole thing is absorbed at once or confusion ensues. You might
have noticed that the definitions although specify a lot of things they still leave
some questions lingering behind. What is the point of a release sequence? Why would
a suitably tagged pair of atomic operations **A**, **B** on object **M**
synchronize whit each other if between them there can be any number of
read-modify-write operations? The answers in this case are clearly specified by the
standard in the below two rules both from section [31.4](https://isocpp.org/files/papers/N4860.pdf#section.31.4):

- 2 An atomic operation **A** that performs a release operation on an atomic
object **M** synchronizes with an atomic operation **B** that performs an acquire
operation on **M** and takes its value from any side effect in the release
sequence headed by **A**.
- 10 Atomic read-modify-write operations shall always read the last value (in the
modification order) written before the write associated with the read-modify-write
operation.