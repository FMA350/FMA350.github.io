---
layout: post
title:  "An intro to Dynamic Programming"
author: Francesco
categories: [ Development ]
tags: [ cpp, development ]
language: English
image: assets/images/loretto_chapel_small.webp
description: ""
featured: true
hidden: false
comments: true
---

Brushing up on concepts and learning dynamic programming by doing. Let's get to it!

## The problem

> You are climbing a staircase. It takes n steps to reach the top.
> Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

Can you think of a solution for a staircase with three steps? What about for one with twentytwo?

## On the shoulders of giants
Recently, I was squandering my time on [Leetcode](https://leetcode.com/problems/climbing-stairs/description/?envType=problem-list-v2&envId=dynamic-programming) when I came across the aforementioned exercise. It was clear, clean, and concise, all while requiring the use of dynamic programming for its resolution. These are qualities seldom found together.
It seemed like the perfect opportunity to approach the subject in this blog series. If you're new to dynamic programming or looking for a challenge, I recommend trying your hand at the exercise first before checking out my solution at the end of the article.

But what is dynamic programming exactly?
Dynamic Programming (or DP for short) is a resolution technique for a whole series of programming problems. It might seem a bit like magic at first, but it is actually all based on the following notions:

* The recursion principle: some problems can be resolved by breaking them down into smaller, analogous subproblems.
* Momoization or "the Uncle Scrooge" principle: we never pay twice for the same result. 

DP is important because once we learn when we can make use of it, we can leverage it to turn seemingly insurmountable challenges into simple and fun exercises!

<p align="center">
  <img src="{{ site.baseurl }}/assets/images/uncle_scrooge.jpg" />
</p>
<p align="center">
    <i>Uncle Scrooge would have approved of this technique</i>
</p>

## Recursion with Fibonacci(n)

> One, one, two, three, five...
> 
> Fibonacci

"The Fibonacci sequence is a well-known concept among computer science freshmen, and it is often used to teach the fundamentals of programming and recursion. The following code snippet should be familiar to anyone with basic programming knowledge:"

```c++
int Fibo(int n)
{
    if(n <= 0) return 0;            // bad input condition
    if(n == 1 || n == 2)  return 1; // The first and second numbers of the series are 1 and 1
    return Fibo(n-1) + Fibo(n-2);   // Sum the previous two numbers in the series to obtain N
}
```
Fibo is a *recursive* function that computes the Fibonacci number 'n' passed in input. The result it outputs is correct, but the function is slow. Extemely slow. The complexity grows exponentially with N (check out [this page](https://www.geeksforgeeks.org/time-complexity-recursive-fibonacci-program/#) for details). Can we do better?

Of course! We can improve the performance of the Fibonacci sequence calculation. The main bottleneck in the current algorithm is that it spends most of its time computing sequence numbers only to discard them immediately. If we could find a way to reuse these intermediate results, we could significantly reduce the computation time...

### A faster Fibonacci with DP

Let's use our previous intuitions on Fibo and save partial results.  

```c++
std::map<int,int> solution;

int Fibo(int n)
{
    if(n <= 0) return 0; // bad input condition
    if(solution.find(n) == solution.end())
        solution[n] = Fibo(n-1) + Fibo(n-2);
    return solution[n];
}
// initalize first elements
    solution[1] = 1;
    solution[2] = 1;
```
What is happening? The Fibonacci sequence is still computed recursively, but with one key difference: a global variable is used to store the results of previous calculations. This 'memoization' technique allows us to avoid redundant computations and significantly improve performance. The memoization map is initialized with values of 1 for both keys 1 and 2, which are the first two numbers in the Fibonacci sequence. When a value is requested for the first time, the computation proceeds as before, but now the result is saved in the map for future reference. If the same value is requested again, we can simply look it up in the memoization map rather than repeating the calculation.

Now this *is* fast! Since we solve each sequence number up to N only once, we can approximate a bit and say that complexity is O(N).

It's easy to see the difference when running side by side and comparing execution time:

| **Sequence number** &nbsp;  | **Fibonacci** &nbsp; | **FibonacciFast** &nbsp;  |
| :----: | :----: | :----: |
| `35` | 29ms | 0ms |
| `42` | 832ms | 0ms |
| `47` | 7633ms | 0ms |
| `1000` | n/a | 0ms |
| `1200` | n/a | 1ms |



Note: you can find both code examples as well as a script to compile them on [my github page](https://github.com/FMA350/code_examples/tree/master/Fibonacci).



## One step at the time

Armed with this knowledge of Dynamic Programming, let us review again the exercise's requirements:

> You are climbing a staircase. It takes n steps to reach the top.
> Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

Our approach is now clear: break the main problem into subproblems and memorize the results for later reuse. We can start with some results from intuition, which will come in handy later. Remember, we can climb either 1 or 2 steps.

> I) How many ways can I climb to rung zero?

There are now solutions, as zero is the starting point. Thus we set this to 0
```c++
    solutions.push_back(0);
```
**Note:** this value could have been skipped, at the price of a more complex logic in the loop. I believe the code is clearer (and faster, thanks to fewer branching statements) with it.

> II) How many ways can I climb to rung one?
Trivial, the only way to the first rung is to move 1 step up. Thus, we set:

```c++
    solutions.push_back(1);
```

> III) How many ways can I climb to rung two?
We either climb 2 step in one go, or climb 1 step twice. Thus two solutions:
```c++
    solutions.push_back(2);
```

Ok, now for the second intuition: if I want to reach rung N, I can either move 2 steps up from rung N-2, or move 1 step up from rung N-1.
The solution should now be trivial:

##### **`climbStairs algorithm`**

```c++
int climbStairs(int n){
    std::vector<int> solutions;
    solutions.reserve(n);
    solutions.push_back(0);  // 0 --> no way to climb 0 steps 
    solutions.push_back(1);  // 1 --> only one way to climb 1 step: { 1 }
    solutions.push_back(2);  // 2 --> two ways to climb 2 steps: {1, 1}, { 2 }
    auto current = 3;
    while(current <= n)
    {
        solutions.push_back(solutions[current - 1] + solutions [current - 2]); // N --> sum the solutions for N-2 and N-1
        ++current;
    }
    return solutions[n];
}
```

Do you have any favorite dynamic programming (DP) algorithms? If so, please share them in the comments below! And as always, happy coding!