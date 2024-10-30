---
layout: post
title:  "Variants vs Any - Part 1"
author: Francesco
categories: [ Development ]
tags: [ cpp, software, development ]
language: English
image: assets/images/Cooling-Tower.jpg
description: ""
featured: true
hidden: false
comments: true
---

"Variants, Any types... is this Python or CPP? Let's discover together these powerful C++17 features in this multi-part article."

## Intro: Strongly typed languages
You might have come across the term "strongly typed." This possibly mysterious phrase refers to programming languages, such as C++, Java, or Rust. But what does it mean? Simply put, strongly typed languages enforce type correctness. Every time a variable is used, such as when it is copied, compared, passed to a function, etc., we ensure that its type matches the type that is expected. For instance:

```C++
    int x = 42; // Assign to an integer variable the value 42. 
    x = "nope"; // Assign to an integer variable a string literal, which causes a compile time error in C++
```

In the case of C++, we tell the compiler that we will store integer values in our variable named `x`. The second statement, in which we try to assign a string literal to `x`, will have it complain! Now there are other languages, called "weakly typed," that do not enforce strict type correctness, such as Python or JavaScript. So then, why bother with strongly typed languages at all, one might ask? For a few important reasons: speed, efficiency, readability and sanitation, ease of debugging, among others. Still, there are times when the ability to be more elastic with types can become an advantage, even in a strongly typed language like C++.

## std::variant
Enter the `std::variant`, a templated container available since C++17, which is often portrayed as a better [union](https://dev.to/pauljlucas/unions-in-c-1ojj). `std::variant` allows us to store values that belong to the complete type set. This snippet:"
```C++
    std::variant<int, std::string> x = 42;
    x = "Hello variant!";
```
is now perfectly legal C++. But what can we do with it?

## A faster dynamic polymorphism

As shown above, we can use `std::variant` as a storage for various types (as long as they are known at compile time!). But another cunning use of variants allows us to simulate dynamic polymorphism:
```C++
    struct vA{

        void who(){
            std::cout << "I am variant A \n";
        }
    };

    struct vB{

        void who(){
            std::cout << "I am variant B \n";
        }
    };

    int main(int argc, char*argv[])
    {
        // Dynamic polymorphism with variants
        std::variant<vA, vB> variant;
        std::visit([](auto&v){v.who();},variant);   // Can you guess what this will print?
    }
```  
We have achieved polymorphism without relying on virtual functions from inheritance or composition, but rather by using similar (i.e., the same method signature) structures and `std::variant` in conjunction with `std::visit`.

> To learn more about visit, check out this [site](https://www.cppstories.com/2018/09/visit-variants/).

Anybody still standing after our whirlwind tour of `std::variant` and its polymorphic possibilities? What about `std::any`? We won't have time to dig into it today. So stay tuned for the next segment my friends, and as always happy coding!

You can find the code example on [my github page](https://github.com/FMA350/code_examples/blob/master/variant_and_any/variant.cpp)

