---
layout: default
title: "Quartz Type System"
---

# Quartz Type System

## STILL A WIP

Lets talk about the Quartz type system. [Previously](/2025/12/17/intro-to-quartz.html) I talked a little bit about
what I don't like about C's types system.And since a working type system is pretty damn important for a programming
language, I figured this would be a good first feature to implement.

Now this post might not be the most coherent ngl. I've never developed a type system in my life. 99% of my programming
is done in C and C++ and thats what I'm used too. This post might just come across as me 
thinking out loud. You've been warned lol.

I think first, it's important to really define the pros/cons of C's type system for a good base to work off of.

Let's consider something simple and easy:

`int foo = 10;`

It's quick and easy to read/write. It's obvious what's going on, "integer variable foo has a value of 10".

But is that really good? Lets think for a moment. 

A variable decleration really has three main parts to it:
- a type
- the variable name
- the value of the variable

Which is these parts is the most important to the developer, and others who read the code?
I'm going to argue that the *variable* itself is the most important, followed by the type, then the value.

Why?

When I write code, I declare variables for things that I plan on using multiple times. I'm pretty sure most modern compilers
have flags that can be set to warn about unused variables in code. So I already know when I declare a variable, I am likekly to see it again.

I'm probably not going to be seeing the type of that variable again. Declaring the type is generally one and done. As far as I'm aware, you cannot re-declare a variable to a new type (at least in C and C++). I know type-casting exists, but this doens't change the underlying type of the variable. Type casting just converts the value from one type to the other, not the actual variable itself.

And in the same vain as the type, the value is seen less than once on average since you can declare empty variables to use later like so:

```c
int x = 10;
int y;

y = x * 2;
```

Notice how we see:
- the variable's two times each (4x)
- the types once each (2x)
- and only the initial value once

This is at least my POV on things. I'd be welcome to hear other opinions on how people percieve this topic.

Going back to our original exmpale of

`int x = 10;`
You need to start scanning from left to right until you get to the variable `foo`.
Then you need to look left to see the type `int`. Then you work your way back to the right to the value `10`.
A little innefficient isn't it. 

With this in mind, lets rework the above to put the most important element first.

`foo int = 10;`

Now from a reading perspective, this reads from left to right as "variable foo of type integer has a value of 10"
which is what we were going for.

Ok, we have a starting point that makes logical sense (at least to me)



