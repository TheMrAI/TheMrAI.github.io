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
- [Fluent C++](https://www.fluentcpp.com/): An extremely good blog about C++ features, techniques and in general
explanations.
- [Cᐩᐩ Weekly With Jason Turner](https://www.youtube.com/user/lefticus1): A C++ Youtube channel with weekly informative
videos on new compiler features and tips.

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

### References

There are a surprising amount of [references](https://en.cppreference.com/w/cpp/language/reference).
lvalue, rvalue, xvalue, prvalue, glvalue

### Move semantics

Move semanthics were added in C++11 and provide a convenient way of defining how resources should be moved from one object to
the other, instead of being copied. Moving resources differs from copying in the sense that the object moved from hands over
the ownership to the moved to object. In other words the lifetime of those objects gets transferred, not necessarily the objects
themselves.

This idea can easily be demonstrated by a class maintaining an array allocated on the heap.

```cpp
template<typename T>
class BigResourceOwner
{
public:
    BigResourceOwner(size_t size = 0) : array_of_data_{nullptr}, size_{0}
    {
        array_of_data_ = new T[size];
        size_ = size;
    }

    // BigResourceOwner(BigResourceOwner& rhs) = delete;
    BigResourceOwner(BigResourceOwner const& rhs)
    {
        array_of_data_ = new T[rhs.size_];
        size_ = rhs.size_;
        for(size_t i = 0; i < size_; ++i)
        {
            array_of_data_[i] = rhs[i];
        }
    }

    // BigResourceOwner& operator=(BigResourceOwner const& rhs) = delete;
    BigResourceOwner& operator=(BigResourceOwner const& rhs)
    {
        if(array_of_data_ != nullptr)
        {
            delete[] array_of_data_;
        }
        size_ = 0;
        array_of_data_ = new T[rhs.size_];
        size_ = rhs.size_;
        for(size_t i = 0; i < size_; ++i)
        {
            array_of_data_[i] = rhs[i];
        }
        return *this;
    }

    ~BigResourceOwner()
    {
        if(array_of_data_ != nullptr)
        {
            delete[] array_of_data_;
        }
    }

    T const* get_data() const
    {
        return array_of_data_;
    }
    size_t get_size() const
    {
        return size_;
    }
private:
    T* array_of_data_;
    size_t size_;
};
```

The class is not very useful, but aside from that it demonstrates some points clearly. It allocates a predefined size of memory
to store **T** type elements in. Depending on the type of **T** this can be a very costly operation, which we may not wish to pay.
This is the exact reason the move operator was introduced. By defining it appropriately we can elegantly sidestep the cost of
allocating new memory and then copying all the elements into it. Instead the moved to class can just take ownership from the moved from
one.

```cpp
template<typename T>
class BigResourceOwner
{
public:
    BigResourceOwner(size_t size = 0) : array_of_data_{nullptr}, size_{0}
    {
        array_of_data_ = new T[size];
        size_ = size;
    }

    // BigResourceOwner(BigResourceOwner& rhs) = delete;
    BigResourceOwner(BigResourceOwner const& rhs)
    {
        array_of_data_ = new T[rhs.size_];
        size_ = rhs.size_;
        for(size_t i = 0; i < size_; ++i)
        {
            array_of_data_[i] = rhs[i];
        }
    }

    // BigResourceOwner(BigResourceOwner&& rhs) = delete;
    BigResourceOwner(BigResourceOwner&& rhs) noexcept
    {
        std::swap(array_of_data_, rhs.array_of_data_);
        std::swap(size_, rhs.size_);
    }

    // BigResourceOwner& operator=(BigResourceOwner const& rhs) = delete;
    BigResourceOwner& operator=(BigResourceOwner const& rhs)
    {
        if(array_of_data_ != nullptr)
        {
            delete[] array_of_data_;
        }
        size_ = 0;
        array_of_data_ = new T[rhs.size_];
        size_ = rhs.size_;
        for(size_t i = 0; i < size_; ++i)
        {
            array_of_data_[i] = rhs[i];
        }
        return *this;
    }

    // BigResourceOwner& operator=(BigResourceOwner&& rhs) = delete;
    BigResourceOwner& operator=(BigResourceOwner&& rhs) noexcept
    {
        std::swap(array_of_data_, rhs.array_of_data_);
        std::swap(size_, rhs.size_);
    }

    ~BigResourceOwner()
    {
        if(array_of_data_ != nullptr)
        {
            delete[] array_of_data_;
        }
    }

    T const* get_data() const
    {
        return array_of_data_;
    }
    size_t get_size() const
    {
        return size_;
    }
private:
    T* array_of_data_;
    size_t size_;
};
```

#### The rule of five

#### Copy elision

### Lambda functions

[Lambda functions](https://en.cppreference.com/w/cpp/language/lambda) are perhaps the most important of features in the C++11 standard.
They allow the user the specification of arbitrary functions and the ability to store and hand over those in a strongly typed manner.
They are described as [closures](https://en.wikipedia.org/wiki/Closure_(computer_programming)) in the documentation, which is somewhat misleading.
They do capture variables from the scope they are constructed in, but in typical C++ manner, whether those values will still exist by the time
the closure is executed is solely the implementers responsibility.

The format of a lambda function is deceptively simple. It consists of a capture block **[]**, a list of parameters **()** and the body of the
function **{}**.
```cpp
[](){};
```

The capture block is where variables can be captured from the enclosing scope, the list of parameters should be thought of as function arguments
and the body will be the function itself.
There is really no need to specify the lambda's return type, but it can be done by a trailing return type.

```cpp
int limit{5};
auto smaller_than = [&limit](int const& rhs) -> bool {
    return rhs < limit;
};

std::vector<int> input{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
input.erase(std::remove_if(input.begin(), input.end(), smaller_than), input.end());
```

The above example demonstrates their power. First we define what is our predicate, in this case we are looking for values smaller than five. Then
**std::remove_if** is called, which removes the values from the vector in exactly big O of N, by moving invalid values to the back, then returns the
iterator pointing to the first value that has satisfied the criteria. To completely eradicate the objects in the vector a final **erase** is called
which executes in big O of N.
In that line of code, the vector is cleared from unimportant values in big O of N time. We couldn't really write much of anything that was either
faster of shorter to scribble down.

Without dwelling much in the details of why and how they do what they do, it is enough to know that they enable all programmers to use the STL algorithms
in convenient manner. When writing C++ code, every line you don't have to write is a win for you, your team, your company in terms of time. Which is
a humongous plus for a language where writing anything down is very cumbersome.

The overwhelming majority of STL algorithms (perhaps all of them, but saying this is easier and definitely correct) expect one type of callable or other
to do their work.

- [std::transform](https://en.cppreference.com/w/cpp/algorithm/transform): Applying a Unnary operator to all elements and putting the results into another range.

```cpp
std::string s("hello");
std::transform(s.begin(), s.end(), s.begin(),
                [](unsigned char c) -> unsigned char { return std::toupper(c); });
```

- [std::find_if](https://en.cppreference.com/w/cpp/algorithm/find): Find an element satisfying the given Unary predicate.

```cpp
std::vector<int> v{1, 2, 3, 4};
auto is_even = [](int i){ return i%2 == 0; };

auto result3 = std::find_if(begin(v), end(v), is_even);
```

- [std::sort](https://en.cppreference.com/w/cpp/algorithm/sort): Sorting a range of elements according to the given comparator.

```cpp
std::array<int, 10> s = {5, 7, 4, 2, 8, 6, 1, 9, 0, 3};
struct customLess {
  bool operator()(int a, int b) const { return a < b; }
};
std::sort(s.begin(), s.end(), customLess);
```

More on them at: [Think of Function Objects as Functions Rather Than Objects](https://www.fluentcpp.com/2017/10/20/think-function-objects-functions-rather-objects/)

Behind the curtains a lambda function isn't really anything special, but a structure implemented with a constructor and a functions call operator.
The following example demonstrates what a lambda function really is and how much code it saves from the implementer/reader.

```cpp
auto execute(std::function<int(int)> method){
    return method(3);
}

int main()
{  
    int local_x = 5;
    auto add_x = [local_x](int number)
    {
        return number + local_x;
    };
    std::cout << execute(add_x);

    struct add_x_functor
    {
    public:
        add_x_functor(int x): x{x}
        {} 
        
        int operator()(int number){
            return number + x;
        }
    private:
        int x;
    };
    auto functor_instance = add_x_functor(local_x);
    std::cout << execute(functor_instance);
    return 0;
}
```

### All about ownership

Since C++11 there are two big families of pointers in C++. One is called **raw pointer/non-owning pointer** the other
is **owning pointer**. They represents completely different things.

A **raw pointer** is called as such, because it does not represent ownership of the resource it is pointing to.
It is just simply pointing to it. For this reason in a modern C++ code when a function returns a raw pointer
it should be assumed by the calling party that the memory location pointed to by the pointer has it's
lifetime handled by some other code.

```cpp
Resource resource;
auto* a_pointer = resource.give_me_pointer();
// working with a_pointer
// don't have to call delete/delete[] on a_pointer
```

An owning pointer on the other hand explicitly states that it has ownership of the memory location it is pointing to and
guarantees that upon it's destruction it will free that memory location.

There are 3 types of owning pointers in C++:

- [std::unique_ptr](https://en.cppreference.com/w/cpp/memory/unique_ptr)
- [std::shared_ptr](https://en.cppreference.com/w/cpp/memory/shared_ptr)
- [std::weak_ptr](https://en.cppreference.com/w/cpp/memory/weak_ptr)

**std::unique_ptr** is the replacement for the old **new-delete/new[]-delete[]** pairs and represents singular ownership.
Only the constructed instance is responsible for owning the resource and nobody else. On it's construction it takes ownership,
on it's destruction it releases the memory location. That is all. No need to handle exceptions and freeing the memory by hand or
making sure in general that on all code paths the resource is released.
In fact from C++14 one should not even have to write down the **new** keyword at all as the handy **make_unique** function was
created as a replacement.

```cpp
auto owned_resource = std::make_unique<int>(3); // equivalent to, but much safer
auto owned_resource = std::unique_ptr<int>(new int(3));
```

To support this idea further a **unique_ptr** cannot be copied only moved! A copy could not be reasonable defined. Which object
would hold the responsibility of freeing the resource after the copy? Because of this fatal flaw was the **auto_ptr** deprecated in
C++11 and subsequently removed in C++17.

**std::shared_ptr** and **std::weak_ptr** represent shared ownership of a resource, but in a slightly different way.
**std::shared_ptr**s own a single resource together. Upon the creation of the first **std::shared_ptr** instance the resource gets allocated, but
it will only get destroyed when the final **std::shared_ptr** pointing to it is destroyed as well.

```cpp
auto user_one = std::make_shared<int>(3);
auto user_two{user_one};
auto user_three = user_one; // all share the same 3
```

**std::weak_ptr** is a little bit different. It shares the ownership, but it will not interfere in the lifetime of the shared resource until it
is upgraded to a **std::shared_ptr**. Until this upgrade occurs the caller cannot access the resource pointed to by the **std::weak_ptr**. As the
**std::weak_ptr** hasn't yet declared participation in the ownership, it cannot know the resource still exists and it will not provide access to it.
This slight modification to the behavior is necessary to being able to properly define the lifetime of objects when they are in a circular dependency.

> As a result of the above the following can be considered as a semantical error in code:
- passing **std::shared_ptr** as an lvalue reference to any scope
The compiler does allow it, but it is like saying "Here I will give you an apple, now it is yours, but it will stay in my pocket."

### Iterators

### The STL

#### Algorithms

#### Containers

#### Chrono

#### Filesystem

#### Threads
