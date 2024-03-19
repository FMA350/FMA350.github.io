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


## What is a partial function application and why is it different that currying?

> "Partial functions applications are oftern confused with currying. I know because I always confused the two!"

Partial function applications are an interesting but often misunderstood concept of theoretical Computer Science.
One might be forgiven for thinking that they are seemingly too abstract to find practical use in any real-world application. 
This though could not be further from the truth.
In fact, I will argue that if you code in C++, you already used partial function applications, maybe unknowilingly. And if you do not know about them, you are doing yourself and your code a great disservice.

But what are these partial functions anyway? Simply put, it is a way to "fix" some or all of the arguments of a function, thus generating a new function that takes in fewer arguments but produces the same result.
More formally, given a **function F, of arity N**, we can use a partial function application to generate an equivalent **part(F) with arity M, where 0 <= M < N**.

Currying is a kindred but distinct concept to partial function applications, as it involves generating a number of sequential functions of arity one. But that's a story for another time.


## Did anybody say methods?

Ok, you are probably wondering, how could this be ever useful to me? When would I want to fix an argument?
Let me give you a very practical example.

Imagine I have a program that makes use of callbacks:

``
std::function<bool(int)>cb;
``

Now, I could assign to this variable a lambda-function, or a normal global function such as this one:

``
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
``
So far, so good.

Enter methods in C++. And yes, before you lose your mind, I know the word "method" is technically not correct within the C++ universe. It is never mentioned in the standard, and it would be more correct to say *member function*. Whatever the jargon one chooses to use, we are talking about functions associated with a class or struct.

These are not the same as global functions. In fact, if when try to do this:

``
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
``

All hell breakes lose. Our friendly compiler points us in the right direction:

> main.cpp:15:12: error: invalid use of non-static member function ‘bool MyStruct::isZero(int)’
>   15 |     cb = s.isZero;
>      |            ^~~~~~

So what is happening here? Methods or member functions hide a secret. 
Unbeknownst to the junior developer, the compiler is modifying the signature of the method by adding an implicit raw pointer to the class type. So while we wrote the function as

`bool isZero(int x)`

What we are really creating looks more like the following:
`bool isZero(MyStruct* this, int x)`
This is the reason you can use ***this*** in your methods, and additionally the reason why you cannot assign this member function to our cb object.


## std::bind to the rescue 

std::bind is the CPP way of creating partial functions. Indeed, we want to *fix* the value of this, and in the process obtain a function that can be assigned in the cb variable.

Let's do just that:

``
    MyStruct s;
    cb = std::bind(&MyStruct::isZero, &s, std::placeholders::_1);
``

What is happening? bind is returning a function equivalent to isZero but with the first argument (MyStruct *this) set to the existing object. Afterwards, we use std::placeholds to indicate that the following argument remains as is. We are not going to fix it.

Voila', we fixed our callback.

## Conclusions

I have seen developers with years of experience getting around this problem by creating static functions with calls to singleton objects, or other horrible spaghettified solutions. I say no more! With the power of partial function applications and what was hopefully a somewhat clear and concise explanation, you too can improve you callback filled codebase.

Happy refactoring!