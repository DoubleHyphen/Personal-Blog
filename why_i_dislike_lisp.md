# LISP is just not for me: A retrospective

![For me, it's olives. For you it's probably something else:  
A long time ago: “Yuck. Olives taste bad.”  
A year later: “Maybe black olives taste better—Nope, gross…”  
A year later: “I'll bet they taste good on pizza—Ugh, does anyone want this slice?”  
A year later: “I wonder why I hate the taste of olives—Yup, I remember now…”  
A year later: “I bet I was just fussy when I was younger—Nevermind, sick…”  
A year later: “I won't let such a small thing defeat me like this—Changed my mind, I give up…”  
](http://www.thedoghousediaries.com/dhdcomics/2010-07-26-f515963.png)

I really, really tried to like LISP. Like the Doghouse Diaries comic above, I failed in every single one of my attempts. 

Having by now given up completely, I thought I'd write an article to detail my thoughts on the issue.


## The strengths of LISP

In the interest of fairness, let us begin with LISP's biggest strengths.

### Historic advantages
A very significant part of LISP's appeal is just _how early_ it was with some features that we today take for granted. To quote Wikipedia:
> As one of the earliest programming languages, Lisp pioneered many ideas in computer science, including tree data structures, automatic storage management, dynamic typing, conditionals, higher-order functions, recursion, the self-hosting compiler,[11] and the read–eval–print loop.[12]

Conditionals! LISP invented the `if`-`then`-`else` structure! How neat is that?

One thing on which I'd like to shine the spotlight is that LISP is the first language to have made it possible to use the functional programming paradigm. This means that, instead of describing what state should be used and how it should be changed, all a programmer has to do is to describe which computations should be applied to the data in order to produce a given desired result. This can lead to a huge simplification in programming: the fewer moving parts, the less chance for accidental error. For instance, filtering a bunch of numbers so that only even numbers are left can be achieved with the following:

```clisp
(remove-if-not #'evenp '(1 2 3 4 5 6 7 8 9 10))
```

In an age when the only alternatives were FORTRAN and assembly, being able to use functional programming was a gigantic productivity boost.

Regardless, all of those are _historic_ advantages of LISP. To make a fair comparison to current programming languages, we need to explain what advantages it has maintained in the 65 years of its existence.

### Advantages it has maintained to this day
The biggest advantage that LISP has maintained until the current day is its metaprogramming ability. LISP gives you the tools to use its own programs as easily as all other data. This means that, instead of writing the program that's needed to solve your problem, you can write the program that writes the program that's needed, or the program that writes the program that writes the program that's needed, nested arbitrarily deep. Whenever LISP lacks a certain language feature, it can just implement it on the fly out of preëxisting language features.


## Garden-variety disadvantages of LISP

Some of LISP's disadvantages are easy to find, even with a cursory glance. To wit:

* The pervasiveness of prefix notation is not very intuitive (humans are used to writing `a + b` instead of `(+ a b)`)
* There are parentheses absolutely everywhere (indeed, they are the only thing denoting syntax in LISP)
* It's rather slow compared to static languages


“One of LISP's most striking features is that both code and data are represented using the exact same data-structure: the list!”  
“Wait. So there's only one data-structure available for data?”

“Let's quickly make a chess engine! Each piece will be represented using an ASCII letter—”  
“Ah. Stringly-typed data. _Exactly_ what I want from a modern programming language.”

“Actually, LISP is fairly fast for a dynamic language!”  
“Well OK, but ‘fast for a dynamic language’ still means ‘slow’, doesn't it?”

“When you reach LISP enlightenment, you won't even be looking at the parentheses: the formatting and indentation will be giving you all the information you need!”  
“So… the correct thing… would be to have significant white-space instead, right?”

“You never need to declare your data-types in LISP. For instance, if you take something as an argument, it could be a scalar value, it could be a list of values, it could be a string of characters, or even executable code!”  
“Oh, now I see! Just, if you'll excuse me for a moment, I have to run screaming towards the opposite direction…”
