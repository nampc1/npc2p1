---
title: "Building Bitcoin, Part 2 - A DIY Guide to Building Finite Fields"
datePublished: Wed Oct 15 2025 06:38:52 GMT+0000 (Coordinated Universal Time)
cuid: cmgrmeajk000302le03oe71nh
slug: building-bitcoin-finite-fields-impl
tags: programming-blogs, bitcoin

---

### From Theory to Code: Let's Build a Finite Field

In Part 1, we took a journey into the strange, looping universe of **Finite Fields**. We used the Color Wheel analogy to understand why Bitcoin needs a mathematical system that is perfectly closed and predictable. We established the two golden rules: the **"wrapping effect"** of modulo arithmetic and the **"prime imperative"** that ensures our system is consistent.

This guide will serve as your blueprint for translating those mathematical concepts into a working, tested piece of code. We are going to create a class that represents a single element of a finite field and teach it the rules of its universe.

By the end, you will have written your own implementation from scratch, giving you a much deeper understanding of how the engines of crypto work. You can compare your final implementation with mine [here](https://github.com/nampc1/rusty-coin/tree/c1-finite-field).

Let's begin!

### What Are We Building?

We are going to create a class that represents a single element of a **finite field**. A finite field is simply a set of numbers with a prime size where all arithmetic "wraps around". For example, a finite field with `prime = 13` consists of the numbers `{0, 1, 2, ..., 12}`. Our class, `FieldElement`, will represent one of these numbers and know how to perform arithmetic according to the rules of its field.

### What You'll Need 🛠️

Before we start, make sure you have the following ready:

* **A programming language of your choice:** This guide is language-agnostic, so you can use whatever you're comfortable with.
    
* **A Big Integer library:** Cryptography requires numbers far too large for standard integer types. Find a well-supported library for your language (e.g., `num-bigint` for Rust, `BigInteger` for Java, or Python's built-in support).
    
* **Basic familiarity with object-oriented concepts:** We'll be creating a class (or structure), so you should be comfortable with that paradigm.
    

---

### Step 1: Design Your `FieldElement` — A Single Number in a Big World

Before writing any logic, let's design our `FieldElement`. What information must it contain to represent a single number within a specific finite field?

This is a critical concept to grasp: our class will represent **a single element**, not the entire field. Think about how we work with regular numbers in programming. When you declare a variable like `x = 5`, that variable `x` holds a single integer. It doesn't represent the entire, infinite set of all integers.

Our `FieldElement` works the same way. An instance of this class is like a single number. To give it context, it needs to know which "world" or field it belongs to.

* **Your Task:** Define a class or structure named `FieldElement` that holds two pieces of data:
    
    1. A `value`: The actual number this element represents.
        
    2. A `prime`: The prime number that defines the field this element belongs to. This `prime` attribute doesn't store the whole field; it's like a passport, providing context and defining the rules this `value` must follow.
        
* **Pay Attention To:**
    
    * **Big Integers are a Must:** Cryptography uses enormous numbers. Standard 64-bit integers won't cut it. You must use a **Big Integer** library for your chosen language to handle both `value` and `prime`. A Big Integer library overcomes this hardware limitation. It represents a single large number as a list or array of smaller, native integers (like `u32` or `u64`). It then re-implements all the arithmetic operations (`+`, `-`, `*`, `/`, etc.) in software, performing the calculations chunk-by-chunk, much like you learned to do long multiplication on paper in grade school.
        
    * **Efficiency:** This "single element" design is incredibly efficient. A field `F(p)` could have trillions of elements. We don't need to store them all. We only create objects for the specific elements we are working with at any given moment.
        

### Step 2: Enforce the Rules of the Field — Validation is Key

A `FieldElement` is not just any pair of numbers; it represents a mathematical concept with strict rules. Your code must enforce these rules relentlessly to prevent your program from entering an invalid, meaningless state.

* **Your Task:** Whenever you create a `FieldElement`, you must check and validate its conditions.
    
* **Pay Attention To:**
    
    * **The Core Condition:** The `value` must be greater than or equal to `0` and strictly less than the `prime`. If you allow an element like `FieldElement(value=15, prime=13)` to exist, you have broken the mathematical definition of the field `F(13)`, and all subsequent calculations with it will be incorrect.
        
    * **Is it Truly a Field?**: A finite field `F(p)` is only mathematically valid if `p` is a prime number. For a robust implementation, you should also check if the provided `prime` is actually prime. While this can be a computationally expensive check for very large numbers, it's essential for correctness. Many Big Integer libraries provide probabilistic primality tests (like Miller-Rabin) that are fast and sufficient for cryptographic purposes.
        
    * **Handling Failure:** Your code must act as a guardian. If an attempt is made to create an element that violates these conditions, it must fail immediately. Do not let an invalid object be created! The standard way to handle this is to throw an exception, return an error, or use whatever error-handling pattern is idiomatic for your language.
        

### Step 3: Implement Equality Checking

Before we can perform arithmetic, we need a way to determine if two `FieldElement` objects are the same. This is a fundamental operation in its own right.

* **Your Task:** Implement a method to check if two `FieldElement` objects are equal. If your language supports it, this is the perfect place to use operator overloading for the equality (`==`) and inequality (`!=`) operators.
    
* **Pay Attention To:** How do you define equality for field elements? It's not enough for their values to be equal; they must also belong to the same field. The check must return `true` only if **both** the `value` and the `prime` of the two objects are identical.
    

### Step 4: Preparing for Arithmetic — The Golden Rule

Now that we have a way to create and compare valid `FieldElement`s, it's time to teach them how to perform arithmetic. We will implement all the standard operations you'd expect:

* Addition (`+`)
    
* Subtraction (`-`)
    
* Multiplication (`*`)
    
* Division (`/`)
    
* Exponentiation (`pow`)
    

Before you write a single line of arithmetic code, you must internalize the **Golden Rule of Finite Field Operations**:

**You can only operate on elements from the same field.**

* **Your Task:** For every arithmetic method you are about to build, add check: `if` [`a.prime`](http://a.prime) `!=` [`b.prime`](http://b.prime).
    
* **Pay Attention To:** If this check fails, your program must stop and signal an error. Allowing an operation between elements from `F(13)` and `F(17)`, for example, is a mathematical error that would lead to meaningless results. Your code must prevent this.
    

### Step 5: Implement Addition and Subtraction

Now for the fun part: arithmetic! The most important rule for any operation is that it can only be performed between elements of the *same field*.

* **Your Task:**
    
    1. Implement addition (`+`).
        
    2. Implement subtraction (`-`).
        
    3. If your language supports it, use **operator overloading** to make your class intuitive to use (e.g., `c = a + b`).
        
* **Pay Attention To:**
    
    * **Field Check:** The very first thing each method must do is check if the `prime` of both elements is the same. If not, raise an error.
        
    * **Modular Arithmetic:** The result of any operation must "wrap around" the field size. The final value must be the result of the operation **modulo** `prime`.
        
    * **The Subtraction Trick:** When calculating `a - b`, you might run into a problem if `b.value` is larger than `a.value`. In math, this gives a negative number, but our field elements must be positive, and the unsigned Big Integer types we use can't store negative values. The solution is to use the principle of modular congruence. The expression `(a - b) mod p` is mathematically equivalent to `(a + p - b) mod p`. By implementing this as `(a.value + prime - b.value) % prime`, we ensure the number we perform subtraction on is always positive, avoiding programming errors while getting the correct mathematical result. For example, in `F(13)`, `3 - 10` becomes `(3 + 13 - 10) % 13`, which is `6 % 13`, resulting in the correct answer, `6`.
        

### Step 6: Implement Multiplication

Multiplication follows the same core principles as addition.

* **Your Task:** Implement multiplication (`*`) for your `FieldElement` class.
    
* **Pay Attention To:**
    
    * Remember the field check!
        
    * The formula is `(a.value * b.value) % prime`.
        

### Step 7: Conquer Division with Fermat's Little Theorem

Division is the most interesting operation. We can't just divide the numbers. Instead, we redefine division `a / b` as multiplication by an inverse: `a * b⁻¹`. We need to find a number `b⁻¹` such that `b * b⁻¹ = 1` in our field.

Luckily, a 17th-century mathematician has our back. **Fermat's Little Theorem** gives us a direct formula for this inverse: `b⁻¹ = b^(p-2) % p`.

* **Your Task:** Implement division (`/`) for your `FieldElement` class.
    
* **Pay Attention To:**
    
    * **The Algorithm:**
        
        1. Perform the field check.
            
        2. Check for division by zero! The `value` of the divisor cannot be zero.
            
        3. Calculate the exponent for the inverse: `prime - 2`.
            
        4. **Leverage your library for modular exponentiation.** A naive implementation of `(base ^ exponent) % modulus` would first try to calculate a gigantic intermediate number for `base ^ exponent`, which is incredibly slow and memory-intensive. All good Big Integer libraries provide an optimized function (often called `modPow` or similar) that calculates the result efficiently without creating that massive intermediate value. You should always use it.
            
        5. The final result is `(a.value * inverse) % prime`.
            

### Step 8: Implement Exponentiation

Exponentiation (`a^e`) is also a common and useful operation.

* **Your Task:** Implement a `pow` method that takes an exponent as input.
    
* **Pay Attention To:**
    
    * Fermat's Little Theorem gives us another optimization here. Since `a^(p-1) = 1` in the field, we can simplify large exponents. The effective exponent is `e % (p-1)`.
        
    * **Use** `modPow` again! Just like in division, this is a perfect use case for your Big Integer library's modular exponentiation function. It's designed for exactly this scenario and is critical for performance. The calculation should be `value.modPow(effective_exponent, prime)`.
        

### Step 9: Test Your Implementation — Trust, but Verify

You've built your class, but how do you know it's correct? When implementing mathematical concepts, unit testing isn't just good practice; it's essential. A single bug in your arithmetic could compromise an entire cryptographic system built on top of it.

* **Your Task:** Write a suite of tests that verify the correctness of every piece of logic you've written. Don't just test the "happy path"; think about the edge cases and error conditions.
    
* **Special Cases to Consider:**
    
    * **Boundary Conditions:** What happens when an operation results in `0` or `prime - 1`? For example, in `F(19)`, what is `12 + 7`? What is `12 - 12`?
        
    * **Identity Elements:** Does `a + 0` equal `a`? Does `a * 1` equal `a`?
        
    * **Subtraction "Underflow":** Does your subtraction trick work correctly? Test a case where `a < b` in `a - b`, like `7 - 12` in `F(19)`.
        
    * **Fermat's Little Theorem:** Does `a^(p-1)` always equal `1` for any non-zero `a`? This is a great way to test your `pow` method.
        
    * **Error Handling:** Does your code correctly fail when you try to add elements from different fields? What about dividing by zero? Make sure these invalid operations don't succeed silently.
        

By building a comprehensive test suite, you can be confident that your `FieldElement` class is a solid, reliable foundation for more advanced cryptographic structures.

### You've Done It!

Congratulations! You have now designed and implemented a `FieldElement` from first principles. You've tackled Big Integers, enforced invariants, and used a famous mathematical theorem to perform complex arithmetic. This little class is a powerful building block, and understanding it is a huge leap toward mastering the fundamentals of Bitcoin.