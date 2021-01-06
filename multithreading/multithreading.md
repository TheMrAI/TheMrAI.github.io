---
layout: page
title: Multithreading
---

## Warning

Multithreading in C++ quite honestly is nothing less than black magic. To anybody
being unfamiliar with the types of magic in general, black magic simply means that
one gets the most power, but the price is great. To properly utilize multithreading
in C++ one has to fully understand the memory model, the physical hardware and all
the implications that may come with them. None of them are easy tasks, even
separately and surely they aren't combined.

The likelihood of making a tiny mistake, that can non the less cause undefined
behaviors at the exact moments one least expects them, is very high. These
mistakes cause the program to crash quickly if(!) we are lucky. If we aren't then
sadly all bets are off and we should spare a prayer for those souls trying to find
what went wrong. This can be an extremely costly endeavour, as these issues cannot
be debugged. The best one can do to fix them is to fully understand the
interactions between threads and fixing **all** the issues that may come up. All
of them. Not only the possible causes of the current issue but every single one. If
this isn't done properly then at the next unfortunate moment a bug comes up again
and the hunt begins anew. A daunting task indeed, especially if the code wasn't
properly written, tested, documented. Simply, if shortcuts were taken prior then
the price for fixing them will be around 10 fold. That would in practice mean,
there will be no time to fix them and they will be carried as long as the Earth
turns or the application stays in use. Not something anybody would wish for anyone
else they cared for.

For these exact reasons I propose these rules of thumb to anyone contemplating
using multithreading:

1. If there are ready made options, prefer them even if they do not match your
use case perfectly.
2. Read the documentation or write it for the tools you are using/making!
3. Implement while exercising utmost humility regarding your knowledge and level
of skill.

Rule 1 saves you and for your colleagues as well an exorbitant amount of time.
An implementation ready made and documented is much, much less likely to be faulty
than anything that could be written in it's place.

Rule 2 is to read the documentation! No, no really! It's is okay you remember it,
still just open the documentation and double check it before any bug is introduced
because a minor detail was forgotten. Don't just search for answers on stack
overflow blindly and assuming that is how it is. Read the documentation! For
example the standard does not guarantee any atomic type except for
**std::atomic_flag** to be lock free. Thus relying on any other type in your
lock free implementation, without checking it in the code is folly.

Rule 3 is to preserve our sanity. If you decide to stretch the limits of your and
everybody elses cognitive abilities coming after you then it is for the benefits of
all, that includes you too, to write the code in the simplest most straightforward
way possible. No need for fancy shorthands and tricky constructs, instead be a
little more verbose than usually necessary. [Bjarne Stroustrup](https://www.stroustrup.com/quotes.html)
puts this principle as "Don't be (too) clever ... "clever code" is hard to write,
easy to get wrong, harder to maintain, and often no faster than simpler
alternatives because it can be hard to optimize."

## Topics

- [Memory model](memory_model.md)

## Acknowledgments

Most of everything in this section is based on the wonderful book: [C++ Concurrency in Action 2nd Edition - Anthony Williams](https://www.amazon.com/C-Concurrency-Action-Anthony-Williams/dp/1617294691/ref=sr_1_2?dchild=1&qid=1609941541&refinements=p_27%3AAnthony+Williams&s=books&sr=1-2&text=Anthony+Williams). The de-facto book to learn multithreading from.