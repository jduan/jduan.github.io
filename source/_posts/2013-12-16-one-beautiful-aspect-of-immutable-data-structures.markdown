---
layout: post
title: "One beautiful aspect of immutable data structures"
date: 2013-12-16 22:22:41 -0800
comments: true
categories: [Immutable data structures]
---

I stumbled upon this [interview question](http://www.careercup.com/question?id=5361917182869504) the other day.
It's not particularly difficult. However, I started appreciating immutable data
structures more after actually coding up a solution.

<!-- more -->

First, I tried to code it up in my favorite language, Ruby. Here's the solution:

``` ruby
def find(arr)
  num_list = [arr]
  find_recur(arr, num_list)
end

def find_recur(arr, num_list)
  if num_list.size == (2 ** arr.size)
    puts "found numbers"
    num_list.each do |l|
      p l
    end
    exit
  end
  new_arr = shift_array(arr)
  arr0 = add_to_array(new_arr, "0")
  arr1 = add_to_array(new_arr, "1")
  unless num_list.member? arr0
    find_recur(arr0, add_to_array(num_list, arr0))
  end
  unless num_list.member? arr1
    find_recur(arr1, add_to_array(num_list, arr1))
  end
end

def add_to_array(arr, element)
  dup = arr.dup
  dup.push element
end

def shift_array(arr)
  dup = arr.dup
  dup.shift

  dup
end

def pad(str, size)
  "0" * (size - str.size) + str
end

N = 32
size = N.to_s(2).size - 1
N.times do |i|
  arr = pad(i.to_s(2), size).split(//)
  find(arr)
end

```

Here's the consecutive bits found:

    5 bits: ["0", "0", "0", "0", "0"]
    5 bits: ["0", "0", "0", "0", "1"]
    5 bits: ["0", "0", "0", "1", "0"]
    5 bits: ["0", "0", "1", "0", "0"]
    5 bits: ["0", "1", "0", "0", "0"]
    5 bits: ["1", "0", "0", "0", "1"]
    5 bits: ["0", "0", "0", "1", "1"]
    5 bits: ["0", "0", "1", "1", "0"]
    5 bits: ["0", "1", "1", "0", "0"]
    5 bits: ["1", "1", "0", "0", "1"]
    5 bits: ["1", "0", "0", "1", "0"]
    5 bits: ["0", "0", "1", "0", "1"]
    5 bits: ["0", "1", "0", "1", "0"]
    5 bits: ["1", "0", "1", "0", "0"]
    5 bits: ["0", "1", "0", "0", "1"]
    5 bits: ["1", "0", "0", "1", "1"]
    5 bits: ["0", "0", "1", "1", "1"]
    5 bits: ["0", "1", "1", "1", "0"]
    5 bits: ["1", "1", "1", "0", "1"]
    5 bits: ["1", "1", "0", "1", "0"]
    5 bits: ["1", "0", "1", "0", "1"]
    5 bits: ["0", "1", "0", "1", "1"]
    5 bits: ["1", "0", "1", "1", "0"]
    5 bits: ["0", "1", "1", "0", "1"]
    5 bits: ["1", "1", "0", "1", "1"]
    5 bits: ["1", "0", "1", "1", "1"]
    5 bits: ["0", "1", "1", "1", "1"]
    5 bits: ["1", "1", "1", "1", "1"]
    5 bits: ["1", "1", "1", "1", "0"]
    5 bits: ["1", "1", "1", "0", "0"]
    5 bits: ["1", "1", "0", "0", "0"]
    5 bits: ["1", "0", "0", "0", "0"]

This code works. However, I have to make copies of arrays when building new
arrays. (methods: *add_to_array* and *shift_array*)

Then I tried to code it up in Ruby with the help of
[Hamster](https://github.com/harukizaemon/hamster) this time. Hamster is a
collection of immutable data structures written in ruby as a gem.

Here's the new code:

``` ruby
require 'hamster'

def find(arr)
  num_list = Hamster.list(arr)
  find_recur(arr, num_list)
end

# arr is the current number
# num_list is the numbers found so far
# Note: we are using immutable data structures here. Otherwise, we have to
# create copies of arrays when modifying arr and num_list.
def find_recur(arr, num_list)
  if num_list.size == (2 ** arr.size)
    puts "found numbers"
    num_list.each do |l|
      p l
    end
    exit
  end
  arr0 = arr.tail.append(Hamster.list("0"))
  arr1 = arr.tail.append(Hamster.list("1"))
  unless num_list.member? arr0
    find_recur(arr0, num_list.cons(arr0))
  end
  unless num_list.member? arr1
    find_recur(arr1, num_list.cons(arr1))
  end
end

# Prefix "0" to a str given the total size.
def pad(str, size)
  "0" * (size - str.size) + str
end

N = 32
size = N.to_s(2).size - 1
N.times do |i|
  arr = pad(i.to_s(2), size).split(//)
  find(Hamster.list(*arr))
end
```

You can see the code is much simpler. Instead of doing defensive copies of
arrays, I simply do **arr.tail.append(Hamster.list("0"))**. New arrays are
created and old arrays are preserved. The recursive calls can use the new arrays
freely, including "updating" them.
