---
layout: page
title: Memory orders
---

## Memory orders

Memory orderings specify how the memory is accessed for an atomic operation for
the object the operation is affecting and for objects around the atomic
operation. The effects of this mind bending sentence with the prior [definitions](definitions.md) is what makes multithreading work in C++. To understand why, first
let's take a look on the memory ordering tags, the building blocks of higher level
interactions.

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
a previously sequentially consistent atomic operation will be denoted to that of a
pair of memory_order_release (on a store) and memory_order_acquire (on a load).

### [Relaxed ordering](https://en.cppreference.com/w/cpp/atomic/memory_order#Relaxed_ordering)

### [Release-Acquire ordering](https://en.cppreference.com/w/cpp/atomic/memory_order#Release-Acquire_ordering)
