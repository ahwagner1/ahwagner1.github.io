---
layout: default
title: "Quartz Grammar"
---

# Quartz Grammar
As of right now (12/20/2025), Quartz has a working lexer. Not all the tokens have been added, but adding in new tokens
is trivial. That's why I decided to design the type system in the [previous](/2025/12/17/quartz-type-system.html) post.

I didn't make a post about the lexer since lexing is a pretty easy task. It's only really string matching and most developers could 
probably build a lexer on their own in a couple of hours.

With the type system buttoned up, and the lexer infra built, it's now time to start building the parser.

But in order to build the parser, we need to build a *grammar* first. 

For those not aware, a grammar is what defines the syntax and structure of a programming language.

[Here](https://en.wikipedia.org/wiki/Syntax_(programming_languages)#Example:_Lisp_S-expressions) is a quick example 
from wikipedia showing a simple grammar using EBN form. 

We will also be using EBNF for describing our grammar in this post, so if not familiar, I recommend a quick detour to 
familiarize yourself.

## Parsing
I wanted to touch really quickly on the parser I will be building in Silica (the Quartz transpiler). 

Silica will make use of a *recursive descent* parser. This is a top down parser, built from a set of recursive procedures.
Recursive descent has its pros and its cons like everything in life. The main pro, is that it's relativily easy to follow and understand.
It's main con is that it's inneficient. Recursive descent requires recursively calling function until reaching a terminal which means that it's 
not unheard of to call >5 functions just to parse a single terminal (like an number literal). There's ways around this but for now we are going 
to stick to only recursive descent. 

Ok let's go more in depth on what recursive descent actually means. First lets look at CFGs.

I'm assuming your (yes you) are familiar with standard context-free grammar. If not familiar with CFG, this might be a bit harder to follow.

Lets define a simple CFG that will generate strings with an equal number of a's and b's:

```
S -> aSb | bSa | SS | \0
```

Note that I am using *\0* in place of epislon here to represent empty strings

Lets try this grammar out. Lets generate the string "abba". 

- We start at S and then choose the third rule *SS*
- We now have tow non-terminals that need parsing. We parse the first *S* using the first rule
- Now, we have *aSbS*. Lets now parse the last non-terminal in our string using the second rule
- Now we have *aSbbSa*. This is close but we still have two non-terminals to parse. We can parse them by choosing the last rule
- We now technically have *a\0bb\0a* but since *\0* represents the empty string, we can simply show that we created the string *abba*

Ok, that was a gentle intro to context-free grammars. How does this translate to parsing our source code?

We can think of our programs as "text that can be created from the CFG". Essentially we need to define a grammar with rules that can 
generate any possible legal permutation of Quartz code.

Ok, starting to see the connection? Lets now look at what some psuedo C recursive descent looks like. For this I'll be using a simpler grammar:

```
S -> aA
A -> abA | \0
```

This grammar just generates strings of a's and optionally b's. Pretty simple but lets me demonstrate a very simple recursive descent parsing example.

```c
void match(char c) {
    if (look_ahead == c) {
        look_ahead = get_next();
    }
    else {
        printf("ERROR");
    }
}

void S() {
    if (look_ahead == 'a') {
        match('a');
        A();
    }

    return;
}

void A() {
    if (look_ahead == 'a') {
        match('a');
        match('b');
        A();
    }

    return;
}

void main() {
    S();
    if (look_ahead == '\0') {
        printf("Parsing successful!");
    }
}
```

Ok, lets walk through this. I find it easier to follow if we have a string to follow along with.
Lets parse the string "aab".

First we start by entering the function `S()` since it's our starting rule. 
Our first character to parse is *a*. The conditional is true in `S()` so we call `match('a')` which checks if they match, then moves to the next 
character. We return from `S()` and then call `A()`. Inside `A()` we match both 'a' and 'b'. At this point, we have parsed everything in the string.
Now `look_ahead` is pointing towards nothing (aka the empty string or null in this case). So when `A()` recursively calls itself, we exit, then exit
`S(). and in `main` we check to see if we reached the end of the string. We did so parsing was a success.

The back trace would look something like:
- `A()`
- `match('b')`
- `match('a')`
- `A()`
- `match('a')`
- `S()`
- `main()` <- starting point

So that's a high level look into recursive descent parsing. I needed that refresher lol 

## Grammar
With that out of the way, lets start working on the grammar. I took a course in college where we covered context-free grammars and all that 
fun stuff. It's been a long time so bare with me here, I'm slowly re-learning.

Lets start by defining the very high level rules:

```
program              = declaration* ;
declaration          = variable-declaration | function-declaration ;
variable-declaration = indentifier ":" type ["=" expression] ";" ;
function-declaration = "fn" identifier "(" params ")" ":" type block;
```

This is a good starting point. We can now define simple variables and functions. Lets see what the associated Silica code would look like

```c++
class Parser {
private:
    const std::vector<TokenData>& tokens;
    TokenData& currToken;

    void expect(Token token);
    TokenData& nextToken();

public:
    Parser(const std::vector<TokenData>& tokens);
    ~Parser();

    int parse();

    void parseVariableDeclaration();
    void parseFunctionDeclaration();
};
```

I know this isn't to totally understand, but it should be good enough to follow along. The way this will work is that the parser gets initialized
with the raw stream of tokens. 

The "main" function will iterate throught the tokens. Our current rules only allow for variable and fucntion declerations. Let's look at `parse()` 
real quick to see what that looks like:

```c++
int Parser::parse() {
    for (auto& token: tokens) {
        if (token.token == Token::IDENT) {
            parseVariableDeclaration();
        }
        else if (token.token == Token::FN) {
            parseFunctionDeclaration();
        }
    }

    return 0;
}
```

We see that if the token is an `identifier`, we assume that we are looking at the start of a variable decleration and jump into the `parseVariableDeclaration()` function.

Otherwise, if we see the `FN` identifier, we know we are looking at a function declaration. We don't really need to look at the source code for those. For now
we just trust that those will then have a similar structure to `parse` in that it will look at the tokens to determine which other branch to jump into.

`void expect(Token token)` will look to see if the `currToken` data member is equal to the passed token. This lets us check our grammars terminators. If 
the tokens don't match, we have an error and need to alert the programmer. If they do match, we advance to the next token and keep going.

So with the bare bones grammar built, lets expand it a bit to support our full type system.

```
program         = declaration* ;
declaration     = var-declaration | fn-declaration ;
var-declaration = IDENTIFIER ":" type ["=" expression] ";" ;
fn-declaration  = "fn" IDENTIFIER "(" param-list ")" ":" type block;
param-list      = [param ( "," param )*] ;
param           = IDENTIFIER ":" type ;

type            = type-prefix* base-type type-suffix* ;
base-type       = "i8" | "i16" | "i32" | "i64"
                | "u8" | "u16" | "u32" | "u64"
                | "f32" | "f64" ;
type-prefix     = ( "[" expression "]" ) ;
type-suffix     = "*";

block           = "{" statement* "}" ;
statement       = var-decl | return-stmt | expr-stmt | if-stmt | for-stmt ;
return-stmt     = "return" expression ";" ;
expr-stmt       = expression ";" ;

expression      = assignment ;
```

This grammar now should be context-free and will get us pretty far. We mainly expanded on the original grammar from earlier in the post.
We now have rules for supporting all of our types, even the complex ones. `type-prefix` and `type-suffix` can handle arrays and pointers.

We have gotten up to actual assignment with this grammar. Variable assignment is surprisingly tricky. Operator precedence is important and 
these rules will determine operator precedence in Quartz. We all know PEMDAS or one of its variants right? Well how stupid would it be if 
we messed up our grammar rules and now addition happens before multiplication?

Generally, the deeper you go in the grammer, the higher the precedence. 

So expanding the grammar above, we could do something like this:

```
1.  program         = declaration* ;
2.  declaration     = var-decl | fn-decl ;
3.  var-decl        = IDENTIFIER ":" type ["=" expression] ";" ;
4.  fn-decl         = "fn" IDENTIFIER "(" param-list ")" ":" type block;
5.  param-list      = [param ( "," param )*] ;
6.  param           = IDENTIFIER ":" type ;
 
7.  type            = type-prefix* base-type type-suffix* ;
8.  base-type       = "i8" | "i16" | "i32" | "i64"
                    | "u8" | "u16" | "u32" | "u64"
                    | "f32" | "f64" ;
9.  type-prefix     = ( "[" expression "]" ) ;
10. type-suffix     = "*";

11. block           = "{" statement* "}" ;
12. statement       = var-decl | return-stmt | expr-stmt | if-stmt | for-stmt ;
13. return-stmt     = "return" expression ";" ;
14. expr-stmt       = expression ";" ;

15. expression = assignment;
16. assignment = equality ("=" equality)* ;
17. equality   = comparison (("==" | "!=") comparison)* ;
18. term       = factor (("+" | "-") factor)* ;
19. factor     = unary ("!" | "-") unary | primary ;
20. primary    = NUMBER | IDENTIFIER | "(" expression ")" ;
```

With this, we should have a pretty simple grammar built. This should be able to handle simple variable and functions declerations.

Let's walk through these rules and see if we can make a simple Quartz function that adds two 
numbers and then returns the value of the sum.

Lets start with rules 1 and 2. We are declaring a function so we jump to rule 4. 

So far we have 1 -> 2 -> 4. In rule 4, our output now looks like:

```
fn identifier(PARAM_LIST) :TYPE BLOCK;
```

Anything in all caps is a non-terminal, and everything else if a terminal. 

What non-terminal should be expand first? Let's work from left to right. 

Expanding on *PARAM_LIST* has us jump to rule 5. So now our code looks like this:

```
fn identifier([PARAM ("," PARAM)*]) :TYPE BLOCK;
```

This is essentially saying, we could have variable number of parameters inside the function declaration.

For the sake of brevity, I'll expand this in a single step but document the rules as I would see them.

Reminder, so far we have done: 1 -> 2 -> 4 -> 5.

Expanding *PARAMS* all the way to non-terminals has us going on the following path:

1 -> 2 -> 4 -> 5 -> 6 -> 7 -> 8 -> 6 -> 7 -> 8 with the code now looking like this:

```
fn add(x: i32, y: i32) :TYPE BLOCK
```

Since identifiers are terminal, I have replace them with actual names so it's easier to see
what function we are building.


Let's now expand the return type and block. 

I will truncate the sequence in the following steps to make it easier to read. I will include
the previous steps last sequence to make it easier to follow. Here is the udpated sequence
for expanding the return type:

... -> 8 (prev step) -> 7 -> 8 (return type expanded)

And here is the updated sequence for expanding the block:

... -> 8 (return type) -> 11

Now our code looks like this:

```
fn add(x: i32, y: i32) :i32 {
    STATEMENT
}
```

Expanding this to generate a return statement that returns the input params leaves us with:

... -> 11 -> 12 -> 13 -> 15 -> 16 -> 17 -> 18 -> 19 -> 20 -> 19 -> 20

which yields:

```
fn add(x: i32, y: i32) :i32 {
    return x + y;
}
```

So our full sequence for building out the above function looks like this:

```
1 -> 2 -> 4 -> 5 -> 6 -> 7 -> 8 -> 6 -> 7 -> 8 ->
7 -> 8 -> 11 -> 12 -> 13 -> 15 -> 16 -> 17 -> 18 -> 
19 -> 20 -> 19 -> 20
```

You can see how recursive descent can be a bit inneficient now I bet. 

Thats a minimum of 23 function calls to parse that bit of source code.


### Footnotes
[^1]: 
