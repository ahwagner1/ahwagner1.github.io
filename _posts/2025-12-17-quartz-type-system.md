---
layout: default
title: "Quartz Type System"
---

# Quartz Type System

## STILL A WIP

Lets talk about the Quartz type system. [Previously](/2025/12/17/intro-to-quartz.html) I talked a little bit about
what I don't like about C's types system.
And since a working type system is pretty damn important for a programming language, I figured this would be a good first feature to implement.

Now this post might not be the most coherent ngl. I've never developed a type system in my life. I write code professionally in C and Python.
So the C type system is really the only one I know. This post might come across as me typing my thoughts out as they come to me.

You've been warned lol.

I think I want to start be "reworking" C's type system. Starting with what I know gives me a good base to go off of instead of reinventing something from scratch.

Let's consider something simple and easy:

`int foo = 10;`

It's quick and easy to read/write. It's obvious what's going on: "integer variable foo has a value of 10".

But is that really good? Lets think for a moment.

A variable decleration really has three main parts to it:
- a type
- the variable name
- the value of the variable

Which is these parts is the most important to the developer, and others who read the code?
I'm going to argue that the *variable* itself is the most important, followed by the type, then the value.

Why?

When I write code, I declare variables for things that I plan on using multiple times. I'm pretty sure most modern compilers
have flags that can be set to warn about unused variables in code. So I already know when I declare a variable, I am likely to see it again.

I'm probably not going to be seeing the type of that variable again. Declaring the type is generally one and done. As far as I'm aware, you cannot re-declare a variable to a new type (at least in C and C++).
I know type-casting exists, but this doens't change the underlying type of the variable. Type casting just converts the value from one type to the other, not the actual variable itself.

And in the same vain as the type, the value is seen less than once on average since you can declare empty variables to use later like so:

```c
int x = 10;
int y = 20;

int diff = y - x;
```

Notice how we see:
- variables a total of 5 times
- types a total of 3 times
- and initial values only twice

This is at least my POV on things. I'd be welcome to hear other opinions on how people percieve this topic. But for now I'm going to maintain my belief
that the hierarchy of imporatance in variable declerations goes variable -> type -> value.

Going back to our original example of

`int foo = 10;`
You need to start scanning from left to right until you get to the variable `foo`.

Then you need to look left to see the type `int`.

Then you work your way back to the right to the value `10`.

A little innefficient isn't it?

With this in mind, lets rework the above to put the most important element first.

`foo int = 10;`

Now from a reading perspective, this reads from left to right as "variable foo of type integer has a value of 10"

So going left to right, we have the sections of the variable decleration in descending order which is what I think makes most sense.

Ok, so now that we have a starting point, lets keep going.

Let's say we want `foo` to be a different size than `int`:

`foo long = 10;`

Not bad, but please consider the following scenarios:

```c
foo long = 10;
bar long int = 10;
baz long long = 10;
quux long long int = 10;
```

Really verbose isn't it.

`long int` and `long` are identical in C
so are `long long int` and `long long`.

`int` is optional in these scenarios. It's also optional when using `unsigned`.

So why have all of these confusing options in the first place? I'm sure there's a logical reason, but I really don't see the need for this in Quartz.

So let's just always make sure the programmer has to specify the exact size of their variables. Using stddef.h we would then have something like:

```c
foo uint8_t = 10;
bar uint32_t = 10;
baz uint64_t = 10;
```

Much better IMO.

But I'm lazy, writing out the full type, then size, and _t just isn't gonna cut it for me.

So how do we be as lazy as possible? I think we just include the first letter of the type as a prefix and then the size:

```
Integers:
u8 u16 u32 u64
i8 i16 i32 i64

Floats:
f32 f64
```

Putting all of this together, lets see what our types look like so far:

```c
foo i64 = 10;
bar u8 = 'C';
baz f32 = 3.14;
```

Not too bad, quick to write, quick to read as well.

So we have numeric types defined, the only things really remaining are pointers, arrays, and function signatures.

## Pointers

In C we could do something like this to define a pointer:

```c
int foo = 10;
int *bar = &foo;
int* baz = &foo;
```

My only real complaint is that you can do both `type* variable` AND `type *variable` like I did with `bar` and `baz` above.
These are identical. Both behave the same way.

However there is a footgun waiting for the less informed with this syntax. For example:

```c
int* foo, bar;
```

Beginners might see this and think that both `foo` and `bar` are `int*`. This isn't true.

Only `foo` is an `int*`, while `bar` is a regular `int`.

In order to declare both variables as `int*` you have to do:

```c
int *foo, *bar;
```

For this reason, I've been team "asterisk tied to the variable" for my programming career. It just makes things safer.

Lets see what this would look like in Quartz:

```
foo i32 = 10;
*bar i32 = &foo;
*baz i32 = &foo;

// what about multiple pointers:
*bar, *baz i32 = &foo;
```

It's not bad, but it's not my favorite. The style of `*variable` in C is used for dereferencing:

```c
int foo = 10;
int *bar = &foo;

printf("Value of foo: %d\n", *bar);
```

To the untrained eye, the Quartz code could look like we are dereferencing pointers during initialization.
Since C specifies the type first, this problem is sort of moot in C, but materializes in Quartz:

```c
// C
int foo = 10;
int *bar = 10;

*bar = ...;

// Quartz
foo i32 = 10;
*bar i32 = &foo;

*bar = ...;
```

See how at first glance, it could look like the initialization is also dereferencing. I don't like that.
I'm gonna break my own rule and now declare that the asterisk needs to be bound to the type instead of the variable like so:

```
foo i32 = 10;
bar i32* = 10;

*bar = ...;
```

This creates a very clear seperation of init vs dereferencing. *AND* don't worry about the potential footgun C has with pointers, 
Quartz will not have that problem. Rest assured that 

```
foo, bar i32*;
```

will both be of type `i32*`.


## Arrays

Let's look at some C style array declerations.

```c
int numbers[5]; // declares an array of 5 integers, no init
int numbers[5] = {1, 2, 3, 4, 5}; // same as above but with init
int numbers[] = {1, 2, 3, 4, 5}; // same as above but size is inferred
int *ptrs[5]; // array of length 5 storing integer pointers
```

Not too bad, lets rework these to fit the Quartz type style

```
numbers[5] i32;
numbers[5] i32 = {1, 2, 3, 4, 5};
numbers[] i32 = {1, 2, 3, 4, 5};
ptrs[5] i32*;
```

Lets revisit the technique of reading from left to right, like we did at the start.
Reading the first array decleration from left to right would be something like

"variable numbers is an array of length 5 of i32"

Doesn't quite roll off the tounge, does it? To me it would make more sense to read as 

"variable numbers is an i32 array of length 5"

So we end up with something like this:

```
numbers i32[5];
numbers i32[5] = {1, 2, 3, 4, 5};
numbers i32[5] = {1, 2, 3, 4, 5};
ptrs i32*[5] = {1, 2, 3, 4, 5};
```

Looks a bit cleaner this way IMO. The one thing I don't love, it the decleration for `ptrs`. 
It almsot looks like we are multiplying `i32` and `[5]`. This doesn't quite make sense but I could see 
people assuming that this is some sort of array expansion technique. Sort of like

```Python
arr = [0] * 5
# arr = [0, 0, 0, 0, 0]
```

Unfortunately, I don't see an easy way around this. I could do something like `i32[5]*`. This type would read
as i32 array of length 5 pointer which makes it sound like a pointer to the array, not an array of pointers.

Putting the asterisk before the type would look like `*i32[5]` which reads as "pointer i32 array if length 5". 
Not super catch either. I think for now I'm just going to keep the syntax `type(*)[N]`. It's not the best visually, but 
it stays consistent with what I have so far.

## Functions
Function signatures should probably have types associated. Lets look at a C function real quick to get a feel for what 
it could look like:

```c
int add(int x, int y);
```

Lets read it out loud now: "integer add with arguments integer x and integer y"
Once again, we have the type before what matters more. Lets rework to like we've been doing:

```
add(x i32, y i32) i32;
```

This reads as "add with arguments x of type i32, and y of type 32 returns an i32"

Notice how we never say function though. So how do we differentiate functions from variables?
I've sort of been holding this topic until we reached functions so I think it's worth visiting here. 

What I think is th easiest way is to add a keyword that specifies functions. In the spirit of being lazy, I probably won't 
add a keyword to specify a variable. It should be obvious enough with one keyword what is a variable and what is a function.

Now what keyword to use? Well I'm ~~lazy~~ efficient and like typing as little as possible. So how about we add in the keyword `fn`. 
So now our function signatures look something like:

```
fn add(x i32, y i32) i32;
```

This now reads as "function add with arguments x of type i32, and y of type i32 returns i32". 
But wait, what about that ending. I can hear you asking "you just said *returns*, which technically is implied" and I would say thats 
correct. I feel like it's extremely important to be explicit when programming. That floating `i32` at the end of the signature could mean 
anything to someone not familiar with Quartz. So the question now is how do we explicitly specify that the ending type is the return type.

Python does this (optionally) by using the arrow operator `->`. I don't hate this. But at the same time, in C the arrow operator is used 
in dereferencing pointers to structs which is probably what will be used in Quartz as well. Am I okay with reusing an operator for two 
different things? I'm not too sure yet. I'll sleep on it and figure out what I want to do. 



