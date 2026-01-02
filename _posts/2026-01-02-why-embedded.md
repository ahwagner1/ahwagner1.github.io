---
layout: default
title: "Why Embedded?"
---

# Why Embedded?
This post will sort of serve as an intro to why I want to pursue embedded development.
I think it will be nice to look back and see where I started, and what my motivations were at the time.

Before we get into why I want to learn embedded, I think a little background helps set the stage.

In college, I had very little hardware exposure. I took a class where we learned simple circuit design using only the basic logic gates. 
In labs we would design and build circuits using simple TI chips. 

This was my only hands-on experience with hardware during my degree which is a shame IMO. 
We took other classes that were *close* to the hardware (like CPU architecture and assembly programming), but nothing else hands-on.

I really regret not taking more CE or EE classes. At the time I didn't realize how much I liked working close to the metal. 

I was originally a Mech. Engineering major. 
I didn't mind it, but I just didn't love it. Spending hours in AutoCAD designing things wasn't really my jam.
What I did like, was the hands-on work. Building something that you designed is incredibly rewarding.

In my second year, I took a engineering computation class that used MATLAB which my first intro to programming. 

I was hooked; programming just clicked with me. I love problem solving and writing code that ends up controlling something as complex as a computer to solve a problem was so fascinating to me.
Being given essentialy a blank slate, and implementing the solution was so fun.

So with all that said, why do I want to join the embedded field instead of the traditional ambitions of people my age who want to join FAANG, or the latest AI labs? 

For a couple of simple reasons:

### 1. I like being close to the hardware
As I started building more projects, I realized that I much prefer low-level systems programming.
I've been exposed to the typical full-stack developer experience and I was not a fan.
Building a mobile app in React my senior year was not a fun project for reasons I'll talk about in the next section.

So why do I like systems programming? 

I think the answer is that I prefer having complete control over my code. 
I wouldn't say I'm a control freak or perfectionist, I just like being able to truly understand the problem domain.
When relying on libraries for programs, you are at the mercy of the libraries maintainers and their design decisions.
They could release a patch that completely regresses a feature you rely on and then you're stuck.
With systems programming, there is less of this. I generally don't need to import an entire framework to rely on.
Do I write all code from scratch? Absolutely not. But you can bet if I'm importing a library, I've read the documentation and source code to understand how things work.

This is what piqued my interest in embedded development. In terms of developing software, it's about as low as you can go without routing the wires manually on the chips. 
You as the developer have complete control over the entire system. 
I like this, it makes you really understand the problem statment, and think about the entire process.

After all, software is only half of the equation. 

### 2. I don't like modern programming
This boils down to two main points: software development is too abstract, and software isn't optimized.

Lets dive in.

#### Abstraction
I mentioned in the previous section that I worked on a React app for my senior project.
I wasn't a fan of that since a lot of the programming involved me importing other libraries and using those.
For the most part, I wasn't solving problems. I was implementing a design that someone else had come up with.

A major reason I fell in love with programming is that there is (generally) no "right" answer. 
You have creative freedom to solve the problem as you see fit. You can design and build the solution from scratch.
Importing libraries limits your creative freedom.

I understand how important libraries has it's place in larger systems, but I'm drawn more to domains that enable me to have full control.

Unfortunately, these days it seems like there is a library for everything and the solution is to just import XYZ and build as fast as possible.

In my eyes, this is just another abstraction to hide complexity and make iteration faster.
There's nothing wrong with that, but I'm not a fan. Complex problems are challenging, and challenges are fun.
Hiding the challenges by importing a library that does all the heavy lifting for me takes away a lot of the fun.

#### Quality
Chips are the faster they have ever been, and more accessible than ever, and yet so many applications don't run well.

Just to name a few examples of what I mean:
- Windows 11 did not have a great 2025 in terms of bugs. 
I think my favorite just from pure irony is the bug where task manager duplicated itself.

- Discord just recently announced that they were trying out a feature where Discord restarts automatically if it's using more than 4GB of RAM. 
To be fair, this is an experimental feature for an edge case and Discord is working on optimizing memory usage.
But it is fascinating how we can put man on the moon with 2KB of RAM and modern desktop applications can use millions of times more memory.

I know this is probably an apples to oranges comparison, but at the same time it points to a central problem with software these days.

*A lot of software just isn't optimized.*

To me it seems like iteration speed is rewarded more than user experience.
Companies are constantly trying to push the next feature, and to be fair this makes sense since that's how they make money.
Their edge is being faster than their competitors and capturing market share.

I don't like this culture. I don't like using a barely functional product, so why would I want to develop one?

### Embedded
All-in-all, I don't think any other fields of computer science quite fit the niche for what I love to do.

Embedded lets me:
- work hands-on
- solve real problems
- write optimized software

As I learn more, it'll be interesting to come back to this to see what's changed and how far I've progressed

#### So where am I at in my learning so far 
Right now, I've been self teaching myself on some STM32F401RE Nucleo boards. 
Everything is being done bare-metal, no HAL and no RTOS. As I learn more, I'll pick up a HAL and RTOS, but I feel it's important to really understand the fundamentals first.

I've made a light blink, and am currently setting up a project that is teaching me interrupts and UART communication.
Once those are done, I'll start on SPI and I2C. 

So far, embedded has been incredibly challenging. The learning curve is steeper than anything I experienced in college.
Sifting through the manuals/documents to figure out all that needs to be done is a new experience to me.

But I'm loving it. It's hard, but that's what makes it fun and worth doing.


If anyone is reading this and wants to watch the progress, all my projects can be found [here](https://github.com/ahwagner1/embedded)
