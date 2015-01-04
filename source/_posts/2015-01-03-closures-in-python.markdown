---
layout: post
title: "Closures in Python"
date: 2015-01-03 23:05:52 -0800
comments: true
categories: [python]
---

Functional languages have a concept of closure, which is a function that closes
its enclosing environment. In other words, a function can access not only its
local variables, but also the non-local variables (aka free variables) of its
enclosing functions.

Python has support for closures but has its weakness and tweaks.

<!-- more -->

## Functions are lexically scoped
An exmaple shows how lexical scope works.

``` python
def outer():
  name = 'John'
  def inner():
    print('name is %s' % name)

  inner()

# invoke outer
outer()
```

The code should print "name is John". The inner function can "access" the free
variable "name" defined in the outer function.

## Closures in Python are weak
You can only "read" free variables but you can't "change" them. For example:

``` python
def make_counter():
  x = 0;
  def counter():
    x += 1 ##Error, x not defined
    print x
  return counter

count = make_counter();
count()
```

You will see an error of "UnboundLocalError: local variable 'x' referenced
before assignment". This is allowed in languages like Javascript.

There is a workaround though. You can use an object instead of a primitive if
you want to "change" free variables.

``` python
def make_counter():
  x = {'count': 0}
  def counter():
    x['count'] += 1
    print x['count']
  return counter

count = make_counter();
count()
```
The code above should work without any compilation errors. It doesn't look
pretty though.

The good news is Python 3 has better support for this. You can use the
"non-local" keyword to tag a variable. That way you can change it.

``` python
def make_counter():
  x = 0;
  def counter():
    # this works only in Python 3
    nonlocal x
    x += 1
    print(x)
  return counter

count = make_counter();
count()
```

## What is a lambda?

In functional languages, lambdas are like synonym of closures. However, in
python lambdas specifically mean "anonymous functions". Anything you can do with
a lambda can be done with a function. Lambdas are easier to use when working
with higher-order functions.

Lambdas are like functions but with limitations:

* they can only have one expression
* the only expression is what is returned (non explicit return is required)

Let's use examples to show how to use a lambda.

``` python
nums = [1,2,3,4,5,6,7,8,9,10]
print(filter(lambda x: x % 3 == 0, nums))
```

This will print "[3, 6, 9]". The lambda passed to the filter function dictates
the criteria is used to filter numbers. You can achieve the same effect using a
plain old function but it looks a bit clumsier.

``` python
nums = [1,2,3,4,5,6,7,8,9,10]
def multiples_of_3(x):
  return x % 3 == 0
print(filter(multiples_of_3, nums))
```

## Conclusion
Hopefully I've shown Python's take of closures and lambdas. They are not as
powerful as their counterparts in pure functional languages but if they are used
right they are quite convienent.
