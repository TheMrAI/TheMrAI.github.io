---
layout: page
title: Cpp overview
---

## Premise

C++ is an old language with a long history. Designed to be a "close to the machine" language
it doesn't pull any punches. Either the user knows what he or she is doing or everything is lost.
Memory is not saved, CPU cycles burn and the entropy of the universe increases. Development in C++
is a slow and arduous process.
For this reason utmost humility must be exercised at all times. Even if the the language is aimed at
giving the user all the power supported by the machine; the developer should always focus first and
foremost writing correct code.

With the above warnings provided let us see modern C++ features absolutely necessary for writing high
quality code and the reasons behind them. This tutorial does not go into the ticket of things. It
won't describe how anything works in detail, it will rather focus the intent behind a feature and how
does that fit into the whole.

## Useful online tools

- [Compiler explorer](https://godbolt.org/): Compile code examples online on the compiler of your choice
and examine exactly what was generated.
- [Quick C++ Benchmark](https://www.quick-bench.com/): Compare the speed of two simple implementations. Don't
assume one way is faster than the other.

## Features

### Initialization

In C++ the developer is responsible for everything. There is no garbage collector coming and cleaning up
after us. There is only you and the machine. You tell it what should happen and **exactly** that is what will
happen. Whether the result was what we have intended is only up to us. This cannot be stressed enough, and it
should be always at the forefront of our thoughts when designing a class. When will things be made and when
will they get destroyed?

#### RAII - Resource Acquisition Is Initialization

Great article on the topic: [To RAII or Not to RAII?](https://www.fluentcpp.com/2018/02/13/to-raii-or-not-to-raii/)

RAII is a principle that says:

>Resource is acquired in the constructor and released in the destructor.

Example:

An [std::thread](https://en.cppreference.com/w/cpp/thread/thread) represents a thread of execution. On construction
creates a system dependent thread object, which it destroys in a system dependent way on it's destruction.
Simple enough however there is a catch! If the thread is joinable on it's destruction an immediate **std::terminate** is
called crashing the program.
That isn't usually what we want so the thread has to be properly joined before being destroyed.

```cpp
#include <thread>

uint64_t fibonacci(uint8_t num) {
    if(num == 0)
    {
        return 0;
    }
    uint64_t previous{0};
    uint64_t value{1};
    for(size_t i = 1; i < num; ++i)
    {
        uint64_t next_value{value + previous};
        previous = value;
        value = next_value;
    }
    return value;
}

int main()
{  
    std::thread compute{fibonacci, 1000000};
    // Section A 
    if (compute.joinable()) 
    {
        compute.join(); // <- without this we crash
    }
    return 0;
}
```

The above code would suffice to handle the immediate terminate issue if everything goes well, but what should be done
if at **Section A** an exception occurs?

How about we abstract away the responsibility of joining a thread into it's own class and let this new classes destructor
join the thread? This is exactly what [std::jthread](https://en.cppreference.com/w/cpp/thread/jthread) does.

With std::jthread the original example simplifies to:

```cpp
int main()
{  
    std::jthread compute{fibonacci, 1000000};
    // Section A 

    return 0; // <-- join called in the destructor
}
```

### Construction and Initializer list

Most vexing parse:

```cpp
struct Vexing
{
    void some_work(){};
};

int main()
{  
    Vexing v(); // <-- this is a function called v returning Vexing and taking no arguments right?
    v.some_work();
    return 0;
}
```

Solution:

```cpp
    Vexing v{};
```

Initializer list:

```cpp
#include <vector>
#include <iostream>

std::ostream& operator<<(std::ostream& os, std::vector<int> data)
{
    for(auto const value : data)
    {
        os << value << ' ';
    }
    return os;
}

int main()
{  
    std::vector<int> first(2, 3);
    std::vector<int> second{1, 2, 3};
    
    std::cout << "Vector first: " << first << '\n';
    std::cout << "Vector second: " << second << '\n';
    
    return 0;
}
```

Result:

```text
Vector v: 3 3
Vector v: 1 2 3 
```

### Implicit/explicit type deduction

### Const correctness

Until C++11 a constant variable, meant a variable for which the compiler would not allow
any non-const operations.

```cpp
const int dont_change = 5;
dont_change = 3; // <-- compiler error
```

This safety check is the same for functions taking const arguments.

```cpp
void change(int& value)
{
    value = 3; // OK
}

void change_const_ref(int const& value)
{
    value = 3; // <-- compiler error
}

void change_const_value(int const value)
{
    value = 3; // <-- compiler error
}

void change_value(int value)
{
    value = 3; // OK
}
```

Not surprisingly **change** and **change_value** are the only two that will compile. The rest will not as the
compiler is protecting us.

> As a side-note, from the compilers perspective two functions with the same name taking arguments as values and
only differing in their constness are exactly the same, which will result in compiler issues stating function redefinition.
```cpp
void change(int value){};
void change(int const value){};
```
The compiler says: `error: redefinition of 'void change(int)'`

This work similarly for member variables.
```cpp
class changer
{
public:
    void change_value_marked_const() const
    {
        value = 3; // <-- Compiler error, trying to modify member variable in a scope marked as const
    }
    void change_value()
    {
        value = 3; // OK
    }
private:
    int value{0};
};
```

All as expected. There is a caveat, however! No variable marked as **const** is actually constant, only the compiler issues errors, for our convenience,
but this does not mean they cannot be changed. The only thing one has to do is cast their constness away.

Functions:
```cpp
void change(int const& value)
{
    auto& non_const_value = const_cast<int&>(value);
    non_const_value = 3; // <-- It isn't const any more and it changes
};
```
Member variables:
```cpp
class changer
{
public:
    void change_value_marked_const() const
    {
        const_cast<int&>(this->value) = 3; // <-- Happily changes this->value to 3 in a const member function
    }

    int get_value() const
    {
        return value;
    }
private:
    int value{0};
};
```

**const** keyword is really important in reducing the number of programming errors, but as it can be seen
it does not mean constant variables. Moreover after the above demonstrations one should be extra suspicions
whenever they see a **const_cast**.

After **C++11** a new keyword **constexpr** was introduced. By [definiton](https://en.cppreference.com/w/cpp/language/constexpr)
a **constexpr** marked expression can be evaluated at compile time.
This will actually make anything marked with it constant. You will not change these values, regardless of what hack you may employ.
Not from inside the code at least.

### Move semantics

&&
std::swap

The rule of five

return

### Lambda functions

### All about ownership

Raw pointer

Owning pointer

- std::unique_ptr
- std::shared_ptr
- std::weak_ptr

### Iterators

### The STL

#### Algorithms

#### Containers

#### Chrono

#### Filesystem

#### Threads