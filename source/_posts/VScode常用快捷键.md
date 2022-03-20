---
title: Learn you a Haskell for Great Good! 
date: 2022-01-04 10:27:00
tags: Haskell, 函数式编程
---

## What is Haskell?

When you are starting out in Haskell, you need keep these items in your mind:

  1. Purely function programming language: `f(x) => g`, if it is called twice with the same parameters ***x***, it is guaranteed to return the same result ***g***
  2. Lazy
  3. Statically Typed: It means `type inference`, you don't explicitly state their type.

## Run ghc's interactive mode to Get a very basic feel for Haskell

### `Operators`

1. simple arithmetic

  > `+`, `-`, `*`, `/`; if we want to have a negative number, it's always best to surround it with parentheses. like `5 * (- 3)`

  ```bash
  $ ghci
  GHCi, version 6.8.2: http://www.haskell.org/ghc/  :? for help  
  Loading package base ... linking ... done. 
  prelude>
  # +-*/
  $ ghci> 2 + 15
  17
  $ ghci> 49 * 100
  4900
  $ ghci> 5 / 2
  2.5
  $ ghci> 50 * 100 - 4999
  1
  # use parentheses to change it
  $ ghci> 50 * (100 - 4999)
  -244950
  $ ghci> 5 * - 3 # ❌
  $ ghci> 5 * (- 3) # ✅
  -15
  ```

2. Boolean algebra: True, False, `&&`, `||`, `not`

  > equality: `==`, `/=`. what about doing `5 + "llama"` or `5 == True` ? we will get a big error message! Don't do that!

  ```bash
  $ ghci> True && False
  False
  $ ghci> True && True
  True
  $ ghci> False || True
  True
  $ ghci> not False
  True
  $ ghci> not (True && True)
  False
  $ ghci> 5 == 5
  True
  $ ghci> 1 == 0
  False
  $ ghci> 5 /= 5
  False
  $ ghci> 5 /= 4
  True
  # but we can do 5 + 4.0
  $ ghci> 5 + 4.0 # 5 can act like an integer or a floating-point number. 
  9.0
  ```

### `function: prefix, infix`

#### `prefix function`

`prefix function` like `succ function` takes anything that has a defined successor and return that successor. it means it can increase the parameter(like numbers).
also, The functions `min` and `max` take two things that can be put in an order(like numbers).

```bash succ, min, max
ghci> succ 8
9
# min returns the one that's lesser
ghci> min 9 10
9
ghci> min 3.4 3.2
3.2
# max return the one that's greater
ghci> max 100 101
101
# Function applications have the highest precedence, as reflected by the fact that when mixed with other operators
# What that means for us is that these two statements are equivalent.
ghci> succ 9 + max 5 4 + 1
16
ghci> (succ 9) + (max 5 4) + 1
16
#  if we wanted to get the successor of the product of numbers 9 and 10, we couldn't write succ 9 * 10
ghci> succ 9 * 10
100
# It is equivalent to
ghci> (succ 9) * 10
100
# we have to write this statement to get 91
ghci> succ (9 * 10)
91
```

#### `infix function`

> backticks: **`**

```bash
# division
# there maybe some confusion as to which number is doing the division and which one is being divided.
ghci> div 92 10
9
# we can call it as an infix function by surrounding it with backticks.
# it's much clearer
ghci> 92 `div` 10
9
```