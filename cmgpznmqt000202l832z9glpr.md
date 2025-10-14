---
title: "From Scratch: Building Bitcoin, Part 1 - A Journey into Finite Fields"
datePublished: Tue Oct 14 2025 03:14:31 GMT+0000 (Coordinated Universal Time)
cuid: cmgpznmqt000202l832z9glpr
slug: building-bitcoin-finite-fields
tags: programming-blogs, bitcoin

---

This series is my personal quest to go from zero to building a working Bitcoin library from scratch. My guide on this adventure is the fantastic book *Programming Bitcoin* by Jimmy Song. I'll be documenting the entire learning journey right here. Let's see how this goes.

---

## Why Our Journey Begins with Finite Fields

Our first step in building Bitcoin isn't writing code for blocks or wallets—it's a quick trip into a special mathematical universe.

It might seem like a detour, but there's a powerful reason.

> Before we can build the cars, buildings, and machines that exist in this universe, we must first understand the fundamental laws of nature that govern it.

For Bitcoin, these fundamental laws make unbreakable digital signatures possible. To secure a global network where everyone must agree, there can be zero room for ambiguity—no rounding errors or decimals. Every calculation must be exact.

That's where **finite fields** come in. They represent these fundamental laws: a self-contained world of integer-only math where every operation is perfectly predictable. Let's start by understanding the rules of our new reality.

---

## Let's Start With Mixing Paint 🎨

In our everyday world, if you start with a few primary colors, you can create a nearly infinite number of new ones. Imagine you have Red and Yellow paint.

When you mix them, you don't get Red or Yellow back. You create a brand-new color: **Orange**. If you then mix Orange with Red, you get another new color, Vermilion. Every time you combine colors, you can expand your palette. Your universe of colors keeps growing and is essentially infinite.

This is an **open system**.

---

## A New Universe: The Color Wheel 🌈

Now, let's imagine a different universe that operates under a completely different set of rules—a universe designed for perfect predictability.

In this universe, only seven colors exist, arranged in a circle: Red, Orange, Yellow, Green, Blue, Indigo, and Violet. You cannot create any new shades.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1760411154258/b4f7e6e3-94b9-4700-9f88-7cf67576734d.png align="center")

The rules for combining colors are also different. When you "add" two colors, you simply move around the wheel. For example, to calculate **Yellow + Blue**:

1. Find Yellow on the wheel.
    
2. "Blue" is the 5th color, so you take 5 steps around the wheel from Yellow.
    
3. You land on Red.
    

So, in this universe, `Yellow + Blue = Red`.

### The Key Difference: A Closed Loop

This closed-loop system is the entire point.

* The paint universe is **open**—operations can create new results outside the starting set.
    
* The Color Wheel universe is **closed**—no matter what you do, you can never leave the wheel. The result is always one of the original seven colors.
    

This idea of a self-contained, predictable universe with a limited number of elements is what mathematicians call a **finite field**. This is exactly what’s needed for a global system like Bitcoin, where every computer must follow the same unbreakable rules and get the exact same result, with no room for new or unexpected outcomes.

So, the difference is clear: Mixing paint in our world leads to endless possibilities, much like working with the infinite real numbers we use every day. But the Color Wheel, with its limited set of colors and its predictable, looping rules, gives us a clear picture of a finite field. This shift—from an infinite, open world to a finite, closed one—is the fundamental building block for the cryptography we're about to build.

---

## What Exactly is a Finite Field?

A **field**, generally speaking, is an algebraic structure—a set of elements equipped with operations—designed for performing arithmetic.

When we talk about a **finite field** (sometimes called a Galois field, or `GF(p)`), we mean it's a field that contains only a fixed, countable number of elements. Within this finite set, there are specific, predefined rules for **addition** and **multiplication**. For a set to be a true field, it must satisfy the following fundamental laws of arithmetic:

### 1\. The Closure Law (Staying in the Sandbox)

This rule simply ensures that if you start within a set of numbers, you stay within it after performing an operation.

* **Rule:** If you take any two numbers, `a` and `b`, from the set, the results of `a + b` and `a * b` must also be in that set.
    

### 2\. The Identity Laws (Zero and One)

These laws confirm the existence of two special numbers that act as neutral elements for their respective operations.

* **Additive Identity (Zero):** There must be a number, 0, such that adding it to any element `a` leaves `a` unchanged (`a + 0 = a`).
    
* **Multiplicative Identity (One):** There must be a number, 1, such that multiplying it by any element `a` leaves `a` unchanged (`a * 1 = a`).
    

### 3\. The Inverse Laws (Opposites)

These laws ensure that every operation can be completely "undone," which is how subtraction and division are defined.

* **Additive Inverse (Negatives/Subtraction):** For every element `a`, there must be an additive inverse, denoted `-a`, such that `a + (-a) = 0`.
    
* **Multiplicative Inverse (Reciprocals/Division):** For every non-zero element `a`, there must be a multiplicative inverse, denoted `a^-1`, such that `a * a^-1 = 1`.
    

### 4\. The Associative Laws (Grouping)

Associativity guarantees that when you combine three or more numbers, the way you group the pairs does not change the final result.

* **Rule:** `a + (b + c) = (a + b) + c` and `a * (b * c) = (a * b) * c`.
    

### 5\. The Commutative Laws (Order)

Commutativity guarantees that the order in which you combine numbers does not change the result.

* **Rule:** `a + b = b + a` and `a * b = b * a`.
    

### 6\. The Distributive Law (Combining Operations)

This unique law describes how the two operations interact with each other.

* **Rule:** `a * (b + c) = (a * b) + (a * c)`.
    

---

From these core rules, we can derive all other standard operations. For example, **subtraction** is simply adding the additive inverse: `a - b = a + (-b)`. Similarly, **division** is performed by multiplying by the multiplicative inverse: `a / b = a * b^-1`. This elegant property ensures the entire arithmetic toolkit is available and consistent inside our finite set.

---

## The Problem with Normal Math (and How We Fix It)

We've established that a finite field must be a closed system, just like our Color Wheel. But if we take a finite set of numbers—say, all integers from 0 to 6—and apply the arithmetic we use every day, we immediately run into a problem.

Using standard addition, if we calculate `5 + 4`, the result is `9`.

Notice that **9** is not in our original set of `{0, 1, 2, 3, 4, 5, 6}`. We've just "fallen off the wheel." The same happens with multiplication (`4 * 3 = 12`). This violates the single most important rule of our closed system: **closure**. Standard arithmetic will constantly produce numbers that don't belong.

To solve this, we can't use normal operations. We need to define a new kind of arithmetic specifically designed to enforce this looping characteristic. This is where the **modulo operator** comes in, giving us a system often called "clock arithmetic".

---

## The Wrapping Effect: Welcome to Clock Arithmetic

What makes these fields truly "finite" and behave like our Color Wheel? It all comes down to a special operation called the **modulo operator**, which gives finite fields their unique **wrapping effect**.

Imagine a standard 12-hour clock. When you add 5 hours to 9 o'clock, you don't get 14 o'clock. Instead, the clock "wraps around" past 12, and you land on 2 o'clock (`14 = 2 (mod 12)`).

This is exactly how arithmetic works in a finite field. Instead of an infinite number line, our numbers exist only between 0 and `p-1`, where `p` is the size of our field. For example, if our field has `p = 7` (like our seven-segment Color Wheel), our numbers are `{0, 1, 2, 3, 4, 5, 6}`. Any time an operation results in a number equal to or greater than `p`, we simply take its remainder when divided by `p`. This **modulo p** operation ensures our result always "loops back" into our defined set of numbers. Just as we moved 5 steps around our 7-color wheel, in a field of size 7 (`p=7`), adding `5 + 4` gives us `9`. To stay on the wheel, we calculate `9 % 7`, which equals `2`. The result is always wrapped back into our set.

This is why, in a finite field, you can never "leave the wheel." This deterministic, self-contained behavior is absolutely critical for the cryptographic protocols that underpin Bitcoin.

---

## The Prime Imperative: Why `p` Must Be Prime

The size of the set, `p`, is known as the **order** of the finite field. However, there's a crucial condition for `p`: **it absolutely must be a prime number** (a whole number greater than 1 that can only be evenly divided by 1 and itself).

Why prime? Because for a number system to be a true "field," every non-zero element must have a **multiplicative inverse**. This means for any number `a` (that isn't zero), there must be another number `b` in the same field such that `a * b = 1 (mod p)`.

Let's see what happens if `p` is *not* prime:

* **Example: Field with** `p = 6` (not prime) Our numbers are `{0, 1, 2, 3, 4, 5}`. Consider the number 2. Can we find a number `b` such that `2 * b = 1 (mod 6)`?
    
    * `2 * 1 = 2 (mod 6)`
        
    * `2 * 2 = 4 (mod 6)`
        
    * `2 * 3 = 0 (mod 6)`
        
    * `2 * 4 = 2 (mod 6)`
        
    * `2 * 5 = 4 (mod 6)`
        
    
    No matter what we multiply 2 by, we never get 1. So, 2 has no multiplicative inverse. This means division by 2 is impossible, and our system breaks down.
    

This breakdown occurs because 2 and 6 share a common factor. When `p` is prime, every non-zero number `a` will always be coprime to `p` (they share no common factors other than 1), guaranteeing that a multiplicative inverse `a^-1` always exists. This is vital for cryptography and the operations we'll encounter soon.

---

## Conclusion: The Blueprint is Ready

Let's bring it all together. We've journeyed from a simple Color Wheel analogy to a formal understanding of a **finite field**: a self-contained mathematical universe with two crucial properties.

First, it has a **"wrapping effect."** All operations are governed by the **modulo operator (**`% p`), which ensures every result loops back and stays within the defined set.

Second, it follows the **"prime imperative."** The size of the field, `p`, must be a **prime number**. This guarantees that division is always possible, making the system complete and consistent.

With these rules, we've defined a perfect, predictable, and unambiguous world. The blueprint is complete. In the next post, we'll move from theory to practice and translate these mathematical concepts into a fully functional finite field class from scratch.