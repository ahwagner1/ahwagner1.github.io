---
layout: post
title: "Quartz Type System"
---

# Quartz Type System

## STILL A WIP

Lets talk about the Quartz type system. [Previously](/2025/12/17/intro-to-quartz.md) I talked a little bit about
what I don't like about C's types system.And since a working type system is pretty damn important for a programming
language, I figured this would be a good first feature to implement.

Now this post might not be the most coherent ngl. I've never developed a type system in my life. 99% of my programming
is done in C and C++ and thats what I'm used too. This post might just come across as me 
thinking out loud. You've been warned lol.

I think first, it's important to really define the pros/cons of C's type system for a good base to work off of.

I think simple types in C shine. Something like:

`int foo = 10;`

is quick and easy to read/write. It's obvious what's going on, "integer variable foo has a value of 10".

But is that really good? Lets think for a moment. I'm evaluating this topic as an English reader/writer/speaker. 

The main topic of a variable decleration is probably the *variable* itself. Don't get me wrong, the type is just as 
important, but most variables are declared and used multiple times in the code, the type is only really specified once.

With this in mind, the above example makes you read from left to right until you see the variable name `foo`.
Then you need to look left to see the type `int`. Then you work your way back to the right to the value `10`.
A little innefficient isn't it. 

With this in mind, lets rework the above to put the most important element first.

`foo int = 10;`

Now from a reading perspective, this reads from left to right as "variable foo of type integer has a value of 10"
which is what we were going for.

Ok, we have a starting point that makes logical sense (at least to me)



