---
layout: post
title:  "Variants vs Any - Part 2"
author: Francesco
categories: [ Development ]
tags: [ cpp, software, development ]
language: English
image: assets/images/Tjentiste_spomenik.jpg
description: ""
featured: true
hidden: false
comments: true
---

"Jump into the world of std::any, and discover a few interesting things about `type erasure` along the way"

## Types go incognito

In last week's [article](https://fma350.github.io/any_variant_1/), we hinted to an even more "generic" std library creation: std::any.
However what do we mean by that and what problem does it attempt address?
To answer that, we need to take a step back and introduce the concept of `type erasure`.

Ok, let's imagine you are working with the following code: a vector of Base*, containing references to derived objects.
```c++
struct Base{};
struct A : public Base{};
struct B : public Base{};
struct GenericVector
{
    std::vector<Base*> _vector;

    void insert(Base* e)
    {
        _vector.push_back(e);
    }
};

GenericVector c;

c.insert(new A);

c.insert(new B);

```

One issue we encounter here is that by storing values in this vector, we lose information about the types of our objects being inserted. We 
only know they belong to `Base` or some subclass inheriting from `Base`. The only way to recover this information is through downcasting or 
changing approach entirely.

```c++
template<class T>
struct GenericVector
{
    std::vector<T*> _vector;

    void insert(T* e)
    {
        _vector.push_back(e);
    }
};
```

Templates are an excellent solution because they allow us to retain information on the type. Indeed, the compiler will create a new 
definition of the class for each `T` used, replacing it in the process. However, we now face a new issue: our vector no longer accepts 
heterogeneous objects! We cannot insert objects of type `A` or `B`, unless we set `T=Base`. But this would cause information loss, which is 
what we were trying to avoid initially.

Additionally, you might not always have a superclass to rely on, as in the case of predefined classes from other libraries or base types 
(`int`, `float`, etc.).

Eureka! What if we erased or hid the type `T` from the view of the `GenericVector`, but exposed it internally to some container class?

```c++

struct Concept
{

};

template<class T>
struct Model : Concept
{
    Model( const T& t ) : object( t ) {}
    T object;
};

struct Container
{
    std::shared_ptr<Concept> object;

    template< typename T >Container(const T& obj ) :
};

```

We can now use our `Container` struct, along with a vector, to store any type we might think of! We are no longer losing any information 
regarding the type but are instead hiding it, erasing it from `Container`'s knowledge. To retrieve it, we just need to use casting:
```c++
    std::vector<Container> v;

    v.push_back(Container(5));
    v.push_back(Container(std::string("TEST")));

    auto containerInt = (*(int*)v[0].object.get());
    auto containerString = (*(std::string*)v[1].object.get());

```

## A standard container

Here's where `std::any` comes in. It's a generic, templated container, just like the one we created earlier, but with the addition of safe casting.

```c++
    std::vector<std::any> vectorOfAny;

    vectorOfAny.push_back(std::any(5));
    vectorOfAny.push_back(std::any(std::string("TEST")));
    try
    {
        auto anyInt = std::any_cast<int>(vectorOfAny[0]);
        auto anyString = std::any_cast<std::string>(vectorOfAny[1]);
    }
    catch(const std::exception& e)
    {
        ...
    }
 
```

## Conclusions

`std::any` can be used in heterogeneous collection as we seen above, or when implementing reflection, or again when dealing with input of unknown type (void *, cough cough...) in a safe manner. Bewaware though, due to runtime type checking, software where performance is paramount might want to steer clear of this class.

I hope you found this article enjoyable and as always, you can find the code example on [my github page](https://github.com/FMA350/code_examples/blob/master/variant_and_any/any.cpp). Happy coding!