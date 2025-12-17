# Intro to Quartz
Quartz is a transpiled systems programming language, heavily inspired by C and Zig. Silica is the name of the Quartz transpiler.

Why am I doing this? Great question, mainly for learning, and to put cool project on my resume. 
Quartz is also a building block to a much larger project I want to work towards. And I also think that compilers are some of the coolest pieces of software that exist.

To be honest, I'm not even sure that Quartz can technically be considered a systems programming language if it's transpiled and not compiled. 
I'm going to choose to be ignorant for now though and just assume it counts. 
Quartz will be transpiled to C, which will then get compiled using the users preferred C compiler.
The first question I foresee people asking is **why write a transpiler instead of a compiler?** 
This is a great question and I think I have a couple of good reasons for this decision.

1. **Portability**
A lot of languages use LLVM for backend compiler infrastructure. I don't blame them, writing the code-gen logic for different architectures would be a pain in the ass. 
I thought about using LLVM for Quartz but after careful research and consideration, decided against it. 
C can compile to anything. I want Quartz to be able to be used anywhere (hint-hint at the bigger project I mentioned earlier). 
By transpiling to C, I achieve the portability that I want and will eventually need, without having to learn LLVM or writing my own code-gen logic.

2. **Simplicity**
I sort of alluded to this in point 1. Writing custom code-gen logic for a single architecture is an incredibly challenging task. 
Now try multiple. Yeah, not happening. That's way out of scope for what I want to get out of this project. 
I also wanted to avoid using LLVM since learning LLVM would be a pretty time-consuming project on its own. 
By transpiling, I get to focus on designing a useable language which is my main goal for Quartz.

3. **Iteration Speed**
Quartz's features are going to be simple. I should be able to express Quartz ideas in C. So why wouldn't I just transpile to C and iterate quickly?

## Why use Quartz when <insert language here> exists?
Now I am aware that langauages like C++, Zig, Odin, ... exist. Shit, theres probably been 100s of so called "C-killers" and C is still kicking.
I am **not** trying to kill C.
I am **not** trying to compete with the likes of C++, Zig, Odin, ...

My main goal is to design a language that improves upon the pain points **I** have in C. C is my favorite language, but it really shows it's age sometimes.
Now before anyone gets upset about me for my words in this section (developers are incredibly defensive about language choice, see the C/C++ v Rust debate), please
read the disclaimer at the bottom.

## Quartz features
Enough of all of that boring stuff, lets talk Quartz features which I imagine are what most people would be interested in.

Quartz is being designed to mimic the simplicity of C, while solving some of the "problems" C has. 
The main things I consider problems in C are:
- lack of modern, QOL features (e.g. struct methods)
- type system
- seperation of interface and implementation
- macros

Lets talk about each of them a little bit. I'm sure I'll get to each more in depth as I build Silica and write more posts but for now heres a quick look at what I mean
for each of the points I just made.

**Lack of modern, QOL features**
- This is pretty simple. C is old, and the programming language field has advanced since then. Certain things make programming easier without incurring 
performance cost so why wouldn't I want to use them. The main feature I wish C had is allowing struct methods. Being able to do something like this:

```c
struct Point {
    int x, y;

    Point(num1, num2) {
        x = num1;
        y = num2;
    }

    void add(num1, num2) {
        x += num1;
        y += num2;
    }
};
```

in C would make code much more organized compared to what has to happen currently for the same functionality.

**Type System**
This one is sort of a toss up for me. On one hand, this:

`int x = 10;`

isn't too bad. It's quick to type and pretty obvios what `x` *is*.

But, something like this:

```c
const char *prt1;
char const *ptr2;
```

in which both `ptr1` and `ptr2` are both pointers to a const char is ugly. 
I know that if your familiar with the exact rules of `const` in C, this is probably obvious. Lets take a look at something a little more sublte:

```c
unsigned int x = 10;
int y = -1;

if (y < x)
    printf("True!");
else
    printf("False!");
```

this ends up outputting "False!". This is becasue we make the comparison between an unsigned and signed integer. 
Ok, still not terrible, but what about this:

```c
void print_size(int arr[100]) {
    printf("%zu\n", sizeof(arr));
}

int main() {
    int arr[100];
    printf("%zu\n", sizeof(arr));
    print_size(arr); 
}
```

**Seperation of Interface and Implementation**
I'm not sure if this should be called something else I'm not aware of, but this essentially mean header files.
I find making and maintaing both header, and source code files cumbersome. I want a single file for the code I'm writing, not two.

**Macros**
These suck. They can easily add bugs that are hard to find. They also hurt readability IMO.

this will print the size of an `int*` instead of the total size of the array. This happens since passing arrays to functions in C decays to a pointer
the first element of the array. `int arr[100]` as the argument of `print_size` is essentially a lie and this can easily be annoying to someone who isn't 
very well versed in C.



## Disclaimer
I want to make clear that I am building Quartz for **ME**. I do not expect a single other person to use this language. It would be cool it someone did, but I'm not going
into this expecting to be the next big thing. I just want to build a cool project in a topic that I find interesting, and maybe learn a thing or two along the way.
