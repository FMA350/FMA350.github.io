---
layout: post
title:  "Dynamically Linked Libraries in C++ - Part One"
author: Francesco
categories: [ Development ]
tags: [ cpp, development ]
language: English
image: assets/images/trinity-library.jpg
description: ""
featured: false
hidden: false
comments: true
---

Compiling a library can be a daunting and intimidating task. Heck, many developers out there may never even need to learn how to do it, particularly if they primarily use interpreted languages.
But for the rest of us with an interest in compiled languages, such as C/C++, Rust, and even Mojo, understanding libraries is *vital*.

In this article, I want to tackle dynamically linked libraries. By the end of this article, you too won't be able to stop ranting about it!

## What are these libraries anyway?

Let's first take a step back. Before we can load a library at runtime, we need to understand what a library is.
From a high level perspective, libraries are collections of code that can be used by other programs and come packaged in a nice format for easy storage, transfer, and usage.

In C++, libraries come in two flavors: *static* and *dynamic*. [The linux documentation project ](https://tldp.org/HOWTO/Program-Library-HOWTO/static-libraries.html) describes static libraries as collections of object files. A great way to think about them is as an intermediate artifact of the compile process. Static libraries by themselves are useless, and can only be consumed by a linker in the process of generating an executable. The output of this process will contain a copy of the static library in its entirety, which in turn can cause the executable size to swell. In GNU/Linux, these libraries are aptly called archives, and have the suffix *.a*.

Dynamic libraries, on the other hand, are more elegant. They can be loaded by multiple programs at the same time, thus reducing the overall size of the files and effectively using storage more efficiently. Additionally, they do not have to be present at compile time but rather at runtime. In the GNU/Linux world, these libraries are called shared objects and have suffix *.so*.

Dynamic libraries are usually loaded at program startup, but we can delay this step until the program is already in execution, a technique called dynamic linking. Let's see how we can do it.

## Getting our hands dirty

> "Why do I always need to wrap my library functions with C code? And why does my program crash at times when I forget to do it?"

To use dynamic linking, we first need to, well... create a shared object library that will be loaded dynamically!

Let's make the simplest possible library we can:

##### **`Library.hpp`**
```c++
#include <iostream>

void greetings();
```

If this is not your first rodeo, you might have noticed I am skipping a rather important step. I assure you that this is intentional. My aim is purely pedagogical. 
For now, let's save this header file and create the relative source file:

##### **`Library.cpp`**
```c++
#include "Library.hpp"

void greetings()
{
    std::cout << "A dangerous greetings() function in a C++ library \n";
}
```

Are you following along? Perfect! We can now grab our trusty compiler. Personally, I am partial to clang, but we will be using gcc and g++ in this case.
The first step is to compile the source file into an object file:

```bash
g++ -c -Wall -Werror -fPIC Library.cpp
```
What is going on here? Let's break the command down:<br>
**g++**: this is our compiler.<br>
**-c** : this flag tells the compiler not to pass the output to the linker.<br>
**-Wall and -Werror**: we ask the compiler to notify us of every warning and to treat every warning as an error. It's a good habit to have these active to improve the quality of our code.<br>
**-fPIC**: now this is an interesting one. fPIC asks the compiler to generate *Position Indipendent Code*, needed for our shared library.<br>
[Note that you can read more about the flags and commands that can be passed to g++ on its linux man page](https://www.man7.org/linux/man-pages/man1/g++.1.html).

Ok, if all went well, this step should have produced and object file, named Library.o. To turn this into a library, let's do the following:

```bash
g++ -shared -fPIC -o Library.so Library.o
```

Ok, just like before we are:<br>
**g++**: calling our compiler<br>
**-shared**: asking it to create a shared library<br>
**-fPIC**: using matching options as above<br>
**-o Library.so**: indicating the name of the library<br>
**Library.o**: indicating all the object files that will be part of this library (a single one, in this case).


Congratulations! You just created your first shared library fully from scratch! Now, let's create an small application that makes use of it by loading it at runtime.

##### **`main.cpp`**
```c++
#include <dlfcn.h>
#include <string>
#include <iostream>
#include "Library.hpp"

typedef void(*FunctionSignature)(void);

int main(int argc, char* argv[])
{
    void* libraryHandle;
    auto libraryPath  = std::string("./Library.so");
    if(!(libraryHandle = dlopen(libraryPath.c_str(), RTLD_NOW | RTLD_LAZY )))
    {
        std::cerr << "Faild to open " << libraryHandle << "\n";
        std::cerr << dlerror() << std::endl;
        return 1;
    }
    else
    {
        std::cout << "Correctly loaded library  " << libraryPath << "\n";
    }

    auto libraryFunction = reinterpret_cast<FunctionSignature>(dlsym(libraryHandle, "greetings"));
    libraryFunction();
    return 0;
}
```

You can see that we are trying to load our Library.so (which is hopefully in the same folder as our executable or in the LD_LIBRARY_PATH) at line 12.
If all is good and dandy, we "grab" our function by using its symbol at line 23. I then call the function in the following line.

## Cliffhanger

All good, right? Well, not really. Depending on your compiler disposition, line 24 might be either printing a nice output from the library or throwing a SIGSEGV (Address boundary error). Let's try it out:


```bash
g++ -o example main.cpp
./example
```


Why the ambiguity? What is happening in our library? Well, I will give you a hint: the culprit is name mangling. Tune back in for part two, where we will explore what is
happening with our symbol table and discover how to avoid this gnarly problem.

Thank you and happy coding!

You can find the unabridged code example (containing the code from part two as well) on [my github page](https://github.com/FMA350/code_examples/tree/master/dynamically_loaded_libraries).