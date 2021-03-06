---
topic: "Rational: ex01"
desc: "private data members, public constructors, integers from command line arguments, main methods, built in toString method, simple compilation/execution with javac/java"
indent: true
code_repo: https://github.com/UCSB-CS56-pconrad/cs56-rational-ex01
---

# cs56-rational-example/ex01

{% include rational_example_header.html %}


# Overview

In this first article, we will first explain what the `Rational` class is supposed to do, i.e. provide an abstraction for a rational number.     

We'll show a class with a `main` method that can be a simple "driver" class&mdash;one that illustrates how to invoke the constructor
for the `Rational` class, and then invoke one or more of the methods of the object instance.

That main class will illustrate how to get simple integer parameters from the command line via the `args` parameter to main (the equivalent of `argc`/`argv` in C/C++).

We'll also provide a "stub" `Rational` class that has the minimum we need to get started:

* private instance variables for `numerator` and `denominator`

* a default constructor, `Rational()`, that takes no arguments, and
  creates an instance of class `Rational`.  We'll assume that this
  creates an instance of rational that represents the number `1`.

* a constructor that takes arguments for `numerator` and `denominator`

* code to get values for those arguments from the command line, and pass them to the constructor. 

* code that prints out these values using the `toString()` method of the class `Object`

We'll see that the built in `toString()` method is not of much use.  In the next
example, we'll override it to make something more useful.  (Note on overriding: See https://ucsb-cs56-pconrad.github.io/topics/java_overriding_vs_overloading/ )


# Basic description of the Rational class

The Rational class provides an abstraction for a rational number, that is one that has an integer numerator, and a non-zero integer
denominator.  For a more formal definition, consult the [Wikipedia article on Rational Numbers](https://en.wikipedia.org/wiki/Rational_number).

This tutorial will not try to motivate *why* we need a class for rational numbers.   We will assume that there is some need to represent these in a program.   We'll assume that once some Java code has gotten input from somewhere for the numerator and denominator of two rational numbers, it needs to:

* Constructor objects that represent these two rational numbers
* Compute new rational numbers representing their sum, difference, and product.   
* Output those rational numbers in some way to the user.

We'll assume that instances of the class are immutable, i.e. we provide a way to construct objects, and getters for numerator and denominator, but no setters for numerator and denominator.

We'll assume that when instances of Rational are constructed, that if there are any common factors between the numerator and denominator, they are factored out.  That is, the rational is represented internally in reduced form, and the getters for numerator and denominator reflect this.  Example:
* Suppose we create a Rational with `Rational r=new Rational(2,4);` 
* Then, `r.toString()` will return `"1/2"`
* The getters for numerator and denominator will return `1` and `2`, respectively.

We'll assume the following rules for negative numbers: 
* The negative sign is attached to the numerator, and the denominator is always positive.  
* Thus getNumerator() can return either a positive, zero, or negative integer, while getDenominator() always returns a strictly positive integer.

We'll also assume that, as a straightforward way of "outputting" rational numbers, a toString() method suffices, with these formatting rules:
* The representation is simply `numerator/demoninator`, e.g. `3/4`, `-6/7`.   
* We'll also assume that when the denominator is 1, that it should be omitted, e.g. `2`, and `0`, not `2/1` and `0/1`
* We'll assume that the negative sign, when present, always appears in the numerator

We'll leave the "quotient" of two rational numbers as a very straightforward programming exercise.

In the first few examples, not all of these rules may yet be in place.  We'll add those incrementally.  Also, as we proceed through the examples, we may make additional assumptions, set additional goals, and/or place additional restrictions.  Those will be noted in the tutorial that goes with each example.

# Simple compiling and executing with `javac` and `java` commands:

The following summary shows compiling and running the simplest possible way
at a `bash` command line using plain old `javac` and `java`:

```bash
-bash-4.3$ ls
Main.java  Rational.java  README.md
-bash-4.3$ javac Main.java
-bash-4.3$ ls
Main.class  Main.java  Rational.class  Rational.java  README.md
-bash-4.3$ javac Rational.java
-bash-4.3$ ls
Main.class  Main.java  Rational.class  Rational.java  README.md
-bash-4.3$ java Main
Usage: java Main int denom
  int and denom should be integers; denom may not be zero.
-bash-4.3$ java Main 3 4
r1 = Rational@2a139a55
r2 = Rational@15db9742
-bash-4.3$
```

You can also use `javac *.java` which compiles both files at once.

A few things to note:

* We can only *run* a class that contains a `main` method.
* We do not specify the `.class` extension when running.
* We instead, specify the name of the class.
* The `Rational.class` and `Main.class` files remain separate.

    * There is no "linking" phase as in C++/C that combines them into a single program
    * There is a way to combine multiple `.class` files into a single `.jar` file, but this is
        not the same as linking, as explained [here](https://ucsb-cs56-pconrad.github.io/topics/java_jars/)
        
* When we write `System.out.println("r = " + r);` we are trying to do string concatentation on two instances
  of the class `java.lang.String`.   Since `r` is a reference to an instance of class `Rational`, it can't
  be concatenated to a String.  In this case, the JVM will try to invoke a `toString()` method on the
  object to convert it to a String.
* The `toString()` method that gets invoked here is actually part of the class `java.lang.Object`

# Every class (directly or indirectly) `extends Object`

When we write `public class Foo extends Bar {` it signifies that Bar
is the parent class of Foo, i.e. the Foo inherits from Bar.

If we write just `public class Foo`, unlike in C++ where that would
signify that Foo has no parent class, in Java, any time we write this,
it is the same as writing `public class Foo extends java.lang.Object`.

Notes:

* Classes that start with `java.lang.` are part of the Java Language.
* That includes `java.lang.String` and `java.lang.Object`, just to name a couple.
* We don't have to write `java.lang.Object`; for any class that is part of the `java.lang.` package, we can leave off the `java.lang.` prefix, and just write `Object`.


One implication of this is that nearly every instance of every class in Java
has a `public String toString()` method, because:

* If it doesn't have its own, it inherits one.
* It may inherit one from its parent class, grandparent class, great-grandparent class, etc.
* But in the limit, if none of those classes have one, there will always be one in java.lang.Object.
* Ok, there are bizarre corner cases where someone deliberately tries to mess this up, 
    say by declaring a private toString() method and making it inaccessible to us.  But mostly, you can rely on there being one.

The thing is, the toString() method we inherit from java.lang.String is seldom what we want.

For example, for a object of type `Rational` with numerator 4, and denominator 3, as the output of:

```java
   System.out.println("r = " + r); // implicit r.toString()
```

We want to see:

```
r1 = 4/3
```

Instead, we get:

```
r1 = Rational@2a139a55
```

To fix this, we need to override the toString method.  We'll do this
in the next article, [Rational: ex02](/tutorials/rational_ex02/)

<div class="github-preview-only">On website: https://ucsb-cs56-pconrad.github.io/tutorials/rational_ex01/</div>
