---
layout: post
title:  "Easy partial function applications in C++"
author: Francesco
categories: [ Development ]
tags: [ cpp, development ]
language: English
image: assets/images/rope.jpg
description: ""
featured: true
hidden: false
comments: true
---


## What is a partial function application and why it is not currying

> "Partial functions applications are oftern confused with currying. I know because I always confused the two!"

In theoretical computer science, partial function applications are an intriguing but frequently misinterpreted idea. It would be understandable to believe that they are too theoretical to have any real-world use.

However, this couldn't be further from the truth!

Indeed, if you write C++ code, you have probably previously utilized partial function calls, even if you weren't aware of it. 
Furthermore, you are doing a disservice to both your code and yourself if you are unaware of them.

But what are these partial functions anyway? Simply put, it is a means to "fix" some or all of a function's arguments, resulting in a new function that requires fewer arguments to achieve the same outcome.

More formally, given a **function F, of arity N**, we can use a partial function application to generate an equivalent **part(F) with arity M, where 0 <= M < N**.

Currying is a kindred but distinct concept to partial function applications, as it involves generating a number of sequential functions of arity one. But that's a story for another time.


## Did anybody say methods?

Ok, you are probably wondering how could this ever be useful to me? When would I want to fix an argument?
Let me give you a very practical example.

Imagine I have a program that makes use of callbacks:

```
std::function<bool(int)>cb;
```

Now, you could assign to cb a lambda-function, or a normal global function such as this one:

```
bool isZero(int x)
{
    return x == 0;
}

int main(int argv, char* argc[])
{
    std::function<bool(int)>cb;
    cb = isZero;
    return cb(1);
}
```
So far, so good. Let's throw another concept in the mix.

Enter methods in C++. And yes, before you lose your mind, I know the word "method" is technically not correct within the C++ universe. It is never mentioned in the standard, and it would be more correct to say *member function*. Whatever the jargon one chooses to use, we are talking about functions associated with a class or struct. Interestingly, these are not the same as global functions. In fact, if when try to do this:

```
struct MyStruct
{
    bool isZero(int x)
    {
        return x == 0;
    }
};

int main(int argv, char* argc[])
{
    std::function<bool(int)>cb;

    MyStruct s;
    cb = s.isZero;
    return cb(1);
}
```

All hell breakes lose. Our friendly compiler points us in the right direction:

> main.cpp:15:12: error: invalid use of non-static member function ‘bool MyStruct::isZero(int)’
>   15 |     cb = s.isZero;

So what is happening here? Methods or member functions hide a secret. 
Unbeknownst to the junior developer, the compiler is modifying the signature of the method by adding an implicit raw pointer to the class type. So while we wrote the function as:

`bool isZero(int x)`

What we are really creating looks more like the following:

`bool isZero(MyStruct* this, int x)`

This is the reason you can use ***this*** in your methods, and additionally, why you cannot assign this member function to our cb object.


## std::bind to the rescue 

std::bind is the CPP way of creating partial functions. Indeed, we want to *fix* the value of this, and in the process obtain a function that can be assigned in the cb variable.

Let's do just that:

```
    MyStruct s;
    cb = std::bind(&MyStruct::isZero, &s, std::placeholders::_1);
```

What is happening? Bind is returning a function equivalent to isZero but with the first argument (MyStruct *this) set to the existing object. Afterwards, we use std::placeholds to indicate that the following argument remains as is. We are not going to fix it.

Voila', we fixed our callback.

## Conclusions

I have seen developers with years of experience getting around this problem by creating static functions with calls to singleton objects, or other horrible spaghettified solutions. 
*I say no more*! With the power of partial function applications and what was hopefully a fairly clear and succint explanation, you too can improve you callback-heavy codebase.

Happy refactoring!