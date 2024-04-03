---
layout: post
title:  "Dynamically Linked Libraries in C++ - Part Two"
author: Francesco
categories: [ Development ]
tags: [ cpp, development ]
language: English
image: assets/images/giants-causeway.jpg
description: ""
featured: true
hidden: false
comments: true
---

## Where we left off

[In last week's article](https://fma350.github.io/dllibraries-pt1/), we learned about libraries (the C++ kind, obviously!), created our very first shared library, and managed to load it in memory dynamically. Sadly, though, we left off on a sour note. The SO we made had the nasty tendency of crashing at runtime when we tried to call its method. Why?


> She was here on earth to grasp the meaning of its wild enchantment, and to call each thing by its right name. By its right name. 
> 
> [Boris Pasternak]

## Name mangling

The witty and highbrowed readers will have "guessed" that the cause is name mangling, a technique used by C++ and other languages to translate a non-unique function name into a unique identifier. C++ allows for functions with eponymous names if, for instance, the parameters are different, an approach dubbed function overloading. Another case is if the functions reside in different namespaces.
But our build tools still requires unique identifiers, though. The solution? The compiler decorates, morphs, and mangles the names of the functions in the symbol's table. Critically,  languages that do not allow for homonymous functions, such as trusty old C, do not need to resort to these stratagems. Remember this detail, because we will exploit it later.

Let's see an example:

##### **`mangled.cpp`**
```c++
void namesake()
{    
    // Some code
}

void namesake(int x);
{
    // Some other code
}
```
In C++ this code compiles just fine. But if we use a C compiler we are made aware, as expected, that something is not right.

```bash
gcc mangled.cpp
m.c:6:6: error: redefinition of ‘namesake’
    6 | void namesake(int x)
      |      ^~~~~~~~
m.c:1:6: note: previous definition of ‘namesake’ with type ‘void()’
    1 | void namesake()
      |      ^~~~~~~~
```

The compiler complaint is clear: we are trying to redefine a function, a no-no in C.

## Under the (n)microscope

Let's see what is going on under the veil with a great utility called [nm](https://linux.die.net/man/1/nm).
Let's compile our mangled.cpp and see what the symbols inside it look like. You should be familiar with the following compiler commands by now: 
```bash
g++ -c -Wall -Werror -fPIC mangled.cpp
g++ -shared -o mangled.so mangled.o
nm mangled.so
```

The output might look intimidating, but it should not be. What we are seeing is the contents of the symbols' table. And at the very bottom, who do we spot?

```
[...]
0000000000001104 T _Z8namesakei
00000000000010f9 T _Z8namesakev
```

It's our namesake functions; they are barely recognizable!

Hopefully, the issue with our shared library last week is starting to appear obvious. When we attempted to call the library function "greetings", we invoked it with its original function name:
```c++
    auto libraryFunction = 
    reinterpret_cast<FunctionSignature>(dlsym(libraryHandle, "greetings"));
```
but the real symbol in the table looked quite different:
```bash
[...]
0000000000001159 T _Z9greetingsv
[...]
```
Since we did not check whether dlsym returned a valid pointer, we caused a segmentation fault.

Of course, we could have protected against this situation with additional checks, but the fact is that our library would remain unusable without digging with nm into the symbols' table. And I can positively assure you, nobody interested in using your library will want to deal with that.  

## Standing on the shoulders of giants

Ok, C++ compilers will mangle our symbol names, whereas C ones will not. Can we exploit this?
Most certainly, we can! Let's wrap our function call around in an extern "C" statement:

##### **`Library.hpp`**
```c++
#include <iostream>

void greetings();

extern "C"
{
    void safeExp();
}
void safe();
void safe(int);
```

The code for the source file becomes instead:
##### **`Library.cpp`**
```c++
#include "Library.hpp"

void safeExp()
{
    safe();
    safe(1);
}
void safe()
{
    std::cout << "A safe() function in C++\n";
}

void safe(int x)
{
    std::cout << "A safe(int x) function in C++\n";
}
```

The compiler sees "safeExp" as a C function, which does not require its symbol to be modified. The two functions named "safe" are C++ and their symbols will be mangled. But this will not be a problem, as we are going to use safeExp() as a bridge to invoke them.

##### **`main.cpp`**
```c++

#include "Library.hpp"

int main(int argc, char* argv[])
{
    auto safeFunctionName = std::string("safeExp");             // Invokes a "C" function, safe
    // This function is safe as it is wrapped in extern "C" within our Library, and thus its name won't be mangled.
    auto safeFunction = reinterpret_cast<FunctionSignature>(dlsym(libraryHandle, safeFunctionName.c_str()));
    
    safeFunction();
    
    return 0;
}
```

Copacetic, isn't it? We made a C function invoke the other in the library. We could have even had it return an object from the class. All of a sudden, all of the library's methods are available to the main program. The sky is the limit.

I hope you found this article as interesting as I had fun writing it. Thank you for coming along this far, and as always, happy coding!

You can find the unabridged code example (containing the code from part one as well) on [my github page](https://github.com/FMA350/code_examples/tree/master/dynamically_loaded_libraries).