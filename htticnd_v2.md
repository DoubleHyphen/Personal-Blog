# Introduction

Sometimes, the greatest quests for knowledge begin with a simple question. From that question, more and more interesting answers are revealed, until one concludes the quest more knowledgeable and more mature. 

And other times, the quest leaves one more confused than before, knowing even less. This is an example of the latter.

And it all started from one small, seemingly-innocent question, namely the following:


# What is a pointer?

In university, I was taught C and C++. I of course had to know what a pointer is in order to pass, and so it was explained to me: It is merely an address in the computer's memory, that knows what it's supposed to find there. By dereferencing it with the asterisk, you instruct the computer to go to that address and read or write something.

Many years later, I chanced upon the blogs of Ralf Jung and Gankra. “Nay!” said they. “A pointer is no mere address! There is more information we associate with it! We don't care just where it points, we also care where that address **came from**! In fact, there is even that one architecture where two pointers with equal addresses can compare unequal if they **come from** different allocations!”.

At first I didn't pay much heed to this. But slowly, a question started brewing inside my mind: “What does Rust do in ordinary architectures, anyway?”. Finally, “just as a quick experiment just before bed”, I wrote a program that examined just that.

### Cool Bear's Cousin's Ominous Narration:
It would **not** be a quick experiment.


# On to the experiment!

Obviously, two pointers with different addresses will compare unequal; that much is utterly uncontroversial. The primary question, however, is whether two pointers with the same address, _but coming from different allocations_, will compare equal or not. To answer this, we're going to attempt to create two pointers to the exact same address in memory, that have been produced from different allocations. We're then going to compare them and see what Rust declares about their equality.

Now, the most fundamental thing for this to work, is to allocate two different things to the exact same point in memory at two different points in time: that way they'd have the exact same address, but **come from** different origins. This immediately disqualifies any possible usage of the heap; a heap allocation can be placed anywhere in the heap whatsoever and still work. So, what if we used the stack instead?

The neat thing about stack allocation is that it is extremely predictable: you allocate all the variables adjacent to one another. You can use a mental image of putting them one on top of the other, and you can only add or remove elements to/from the top of the stack. This is exactly why this area of the computer's memory is called a “stack”, complete with a “top” and “bottom”[^¹].
[^¹]: Nothing to do with quarks, or, er, other usages of those terms.

This immediately allows us to create two different allocations that point to the same address! Because stack allocations only ever happen at the very top, if we allocate something, save its address, immediately de-allocate it, and allocate something different there, we can be reasonably sure that they will end up in the same address. Good enough for our experiment!

Here is what we are going to do, then:
* Allocate a `u8` with a random value, and save its address.
* Immediately deallocate it, allocate a new one, and save its address again.
* Enable compiler optimisations (a crucial step!)
* Compare those two pointers.

Let's give it a whirl:
```rs
fn main() {
    let a: *const u8;
    let b: *const u8;
    {
        let v = rand::random::<u8>();
        a = &v;
    }
    {
        let v = rand::random::<u8>();
        b = &v;
    }
    println!("{a:?} == {b:?} evaluates to {}", a==b);
}

```
Here's a possible output:
```
0x7ffe6d8ee318 == 0x7ffe6d8ee318 evaluates to false
```
How interesting! It appears that, although both addresses are identical, the comparison returns `false`! Except… what if by accident we print it twice?
```rs
    println!("{a:?} == {b:?} evaluates to {}", a == b);
    println!("{a:?} == {b:?} evaluates to {}", a == b);
```
Output:
```
0x7ffc2bd668a0 == 0x7ffc2bd668a0 evaluates to false
0x7ffc2bd668a0 == 0x7ffc2bd668a0 evaluates to true
```

Wat.

Seriously, wat.

The first time the comparison occurs, it evaluates to `false`. The second, it evaluates to `true`.

Hahaha, isn't pointer comparison silly? Let's cast them to `usize`s so it works correctly:
```rs
    // Rest of program as before.
    let a = a as usize;
    let b = b as usize;
    println!("{a:?} == {b:?} evaluates to {}", a == b);
    println!("{a:?} == {b:?} evaluates to {}", a == b);
```
Output:
```
140732119457488 == 140732119457488 evaluates to false
140732119457488 == 140732119457488 evaluates to true
```

I, uh.

I…

…I think we just broke integer comparison.


# Wat

I guess it's now time to explain how this came to pass.

Before we do that, however, a word of warning: If you're looking for a good ending, you won't find it here. As of the time of this writing, no workable back-end solution has been found. A programmer can side-step the issue by using `black_box(a)` instead of `a`, and that's about as good as it gets.

And now, the explanation. The tl;dr is “The sins of the parents are visited unto the children”; and the parent most pertinent here is C. C is the oldest still-widely-used language that gave its users pointers from the get-go[^²]. However, compared to other languages, it sometimes feels that pointers are _all you have_ in C. As a result,


# When all you have is a pointer…

…everything starts to look like a dereference. C has so many uses for pointers that it's hard to keep track of them all:

Want to return more than one value from a function? Pointers.  
Want a polymorphic function? Pointers.  
Read an argument without modifying it? Pointers.  
Read it AND modify it? Pointers.  
Use arrays?  
Strings?  
Dynamic memory?  
Nullable arguments?  
User-supplied code?  
Iterators?  
Pointers, pointers, pointers, pointers, pointers, pointers.  

This of course makes a programmer sad, because you can't communicate your intentions through your API. A function that takes a pointer might require any of those things, or even more than one, and there's no way to know.

What might come as a surprise is that this makes compilers unhappy too, because they want nothing more than to optimise code, and to do that they need to actually _know things_ about it. To use a pointer correctly, you need to know all of the following:

* Is it usable? As in, is it aligned, non-null, non-dangling? All 3 need to be affirmative if you want to dereference it, so all the other questions are moot if one of those is false.
* Can it be read from? As in, is the memory to which it points initialised?
* Can it be written to?
* Can it be incremented in order to read further along in memory? If so, how far can it go without going out of bounds?
* Can the pointee be deallocated if this specific pointer no longer needs it?
* And, extremely importantly: Are there any other pointers that have access to the same area in memory? If so, what are they allowed to do with it?

Ordinary pointers, on their own, answer precisely none of those questions. And the worst part is that, once you decide to take a raw pointer as an argument, you have to cover every single one of those cases or risk CVEs.

We mentioned C, but what about other languages? A very important part of language design is to decide how you'll approach this issue. Many languages, for instance, solve this by saying “No pointers, ever. We'll copy things for you if need be, and delete them ourselves when they're no longer used.”. This can work fairly well, depending on the use-case. For other use-cases, however, it's the exact opposite of what you want.


## Intermission: When Garbage Collection goes awry

Open parenthesis.

The bargain that GC makes is easily understood: “Throw some more hardware at the problem, and I'll save you some head-aches.”. The edge-cases that can cause more head-aches than they solve, however, are less obvious.

Firstly, the reason for which Discord dropped Go for Rust: Latency. GC frequently takes the attitude that, while it works, any requests made to the program are replied to with “Go away, I'm busy now.”. If you only care about the average latency, this might not be an issue. If you care about the worst-case response time, however, it can be frustrating to realise that you can never have predictable worst-case latencies no matter how much hardware you throw at the problem or how well you structure your program.

Secondly: there are other resources to manage apart from memory! Even if we assume that a “Whoops, you leaked this memory block, here have another one instead” is acceptable, a “Whoops, you leaked this file handle, here have another one instead” is very much not. The usual work-around is to `.close()` the file explicitly… which means that any file handle can remain alive even after closing… which in turn means that no file handle can ever be trusted to be definitely open. Similar problems can arise eg with web sockets, mutex locks, thread handles etc.

Finally, a problem mentioned by Bryan Cantrill: A GC can enable shoddy architecture, and shoddy architecture can easily negate every single benefit that GC has. In his own slightly paraphrased words: “If you have a huge interconnected network of objects that all depend on one another, and ONE left-over closure with ONE reference to ONE of those objects, the entire network is considered live memory and cannot be deallocated. At this point, the garbage collector becomes a mere heap-scanner, and it's scanning the heap just fine, thank you very much!” ( https://www.infoq.com/presentations/os-rust/ )

In a sense… it's a smidge difficult for GC to be both necessary and sufficient. If your architecture is good, life-times are sufficient; if your architecture is shoddy, not even GC can save you.

Close parenthesis.


# What does Rust do?

Rust's answer to the afore-mentioned problems is to give the programmer several different pointer types, each of which has a guaranteed response to all of those questions; if not at compile-time, at least at run-time. For instance, for a `&[T; 2]` the answers are respectively “Yes, yes, no, yes, no, yes”. For an `Option<Rc<RefCell<[u8]>>>`, all of the answers are “maybe, ask me at run-time”[^⁸]. And so on.

[^⁸]: Technically, there are two possible ways why this might not be dereferenceable: It could be `None`, or it could contain an empty slice. Focusing on that would have muddled the point, however.


### Cool Bear's Cousin's Hot Tip:
In safe Rust, not all those questions are orthogonal. To wit:
* Most of the time, if something can be written to, it can also be read from. `MaybeUninit` is quite possibly the only exception.
* Nothing can *ever* be deallocated if there are still other dereferenceable pointers that point to it. 
* _Most of the time_, the phrase “there are other dereferenceable pointers to it” is the logical negation of the phrase “we can write to it”. Exceptions include, for instance, Atomics (because you can write some things through shared references) and types used as tokens (because you can't really write anything in those anyway).

As it just so happens, even LLVM agrees that the answers to those questions are important! Its _first_ edition was completely agnostic about them. But slowly, over its history, it has acquired the necessary vocabulary to express almost all of them! And the first one to appear historically, as early as LLVM 2.1, as if it'd been judged to be the most important…

…was the last one.


# Pointers and provenance

The last question, as it turns out, is important enough that even C has the vocabulary to express it: it's called `restrict` in C and `noalias` in LLVM. Of course, in practice, `restrict` was so rarely used in C that that it took the Rust team *several* roll-backs to fully stabilise all available uses of `noalias` in LLVM, and only in 1.54 was it stabilised for good. I mean, hopefully? I hope I won't jinx it.

### Cool Bear's Cousin's Snarky Observations:
* Yes, that means that `noalias` was broken for _fourteen years_, six of them demonstrably so[^³]. Hush, this is not important right now.
* And yes, it _is_ comical that LLVM is more expressive than C in this sense, thank you for asking.

[^³]: First demonstration I could find was https://github.com/rust-lang/rust/issues/29485#issuecomment-152687156

Anyway, one trick that compilers use in order to gain some semblance of an answer to that question is the following convention: No pointer can ever be used unless it's been given responsibility for a _specific memory portion_, for a _specific amount of time_, depending on the allocation. This is called “provenance”[^⁷], which is Latin for “where did you *come from*‽”. It's a piece of metadata that every pointer has; in the supermajority of cases, it only exists during compile-time.

[^⁷]: I know it's a weird word. I find it helps if I pronounce it with the most obnoxiously thick French accent I can muster. Pʁʁʁʁʁʁʁʁʁʁʁʁovenànce!

So, every pointer is considered to **come from** a specific allocation, and can only be used to access the memory that corresponds to this specific allocation, for the time that this specific allocation is active. As soon as this allocation becomes inactive, all pointers that **come from** it are considered invalid to ever dereference, even if the memory to which they point becomes live again. (The pointers then are said to “dangle”.)

And here is where LLVM's overly-aggressive optimisations come into play: In C and C++[^⁴], a dangling pointer is not merely invalid to _dereference_, it's invalid to _use in any capacity_. In other words, a dangling pointer's value is no longer eg `0x7ffc2bd668a0`, or even `null`; it's `stop bothering me with this nonsense`. Thus, because the pointers were both dangling when we compared them in the above example, _any comparison_ between them is considered by LLVM to be invalid, and therefore returns false.

[^⁴]: For C, I've found conclusive proof. For C++, I found that, at least around 2004, this was UB. However, SCOTI (ie Some C███ On The Internet) said that now-a-days it's implementation-defined, but I can't find a source for that.

Now add to that the fact that, for all architectures except one, provenance is a compile-time concept. Thus at run-time, pointers are basically integers anyway, so when we cast the pointers to integers LLVM decides to just, like… not do that. Hence the appearance of so-called _indeterminate values_ in ordinary integers, and integer comparison becoming non-deterministic. Even evaluating `(a_0 ^ b_0).count_ones() < 1` will evaluate to `false`[^⁵] because, as long as those pointers are dangling, LLVM doesn't have to care about what happens with them.

[^⁵]: To say nothing of
    ```rs
        a.to_ne_bytes()
            .into_iter()
            .zip(b.to_ne_bytes().into_iter())
            .all(|(a, b)| a==b)
    ```
    , which _also_ evaluates to `false`. LLVM's optimisations are nothing if not persistent, apparently.

# But, LLVM really should care!

Yes. You and I think that, and its developers think that, but its technical baggage disagrees very strongly. And it has veto power over the entire thing.

See, for the longest time, outside of C and C++, LLVM never really had to worry about exposing pointer usage externally. That's because, in addition to being quite specialised for those two languages, it only supported very few other languages (Ada, Fortran, C#) that both had pointers and were anywhere remotely main-stream. Even then, all three of those had other older and much more popular implementations. As a result, LLVM settled quite understandably into a comfort zone: Any usage of pointers that was correct _in practice_ in C and C++ was considered correct for every language in every program. This is exactly why nobody knew that `noalias` was broken: because C/C++ programs basically never used `restrict`, it was used very rarely in practice, so LLVM hadn't had to really care about the issue until Rust came along. And Rust has its own opinions on pointers.

In Rust, even when you have a raw pointer, as long as you never dereference it, you can do _anything you want with it_ and it's considered safe[^⁶]. Thus, as far as Rust's semantics are concerned, comparing dangling pointers is just fine, because none of the program's invariants can be broken that way. LLVM happens to disagree, leading to the absolute mess that we stumbled upon at the beginning.

[^⁶]: `add` and `sub` are exceptions, because they give stronger guarantees about their result in exchange for permitting UB.

Finally, we saved the best for last: in Rust, pointer comparison is defined as a comparison _of their addresses alone_. In other words, the program was wrong to output `false` from the get-go, even before the non-determinism reared its ugly head.

# So… now what?

Now, we shake our heads in impotent disappointment, and twiddle our thumbs in wait.

I previously made the prediction that we will see a Rust-side work-around long before we see an LLVM-side solution. Currently, neither appears feasible, so we have no choice but to wait and see.

## PS: In case you hadn't despaired enough already…

…this behaviour can _also_ reliably trigger on GCC. All it takes is to translate the program to C:
```c
#include <stdio.h>
#include <stdint.h>

int main(void) {
    uintptr_t a;
    uintptr_t b;

    {
        int v[2] = {0, 0};
        a = (uintptr_t)&(v[0]);
    }

    {
        int v[2] = {0, 0};
        b = (uintptr_t)&(v[0]);
    }

    printf("%ld ==/^ %ld is %d, %ld\n", a, b, a == b, a ^ b);
    a ^= 1, b ^= 1;
    printf("%ld ==/^ %ld is %d, %ld\n", a, b, a == b, a ^ b);

    return 0;
}
```
This program produces the exact same behaviour, in both GCC and Clang, with both C++ and C compilers, as long as optimisations are enabled. The problem runs very, _very_ deep. (BTW, if this bug has already been reported in GCC, I'd appreciate a link.)

[^²]: FORTRAN and COBOL are older, but got pointers _much_ later than C. PL/I, ALGOL, Pascal and BASIC did have pointers from the get-go, but are no longer in wide use.
