# The three types of programming language complexity


## Introduction

One of the criticisms that's frequently levelled against Rust –not unfairly, might I add– is that it's a complex language. It has lots of syntax, it has weird macros, it makes you clone everything when you just want to get the damn thing to work. All of those are admittedly true.

So why do so many people love it? Why has it been elected most-loved language in the Stack Overflow polls for (as of this writing) eight years in a row? Are we all systems engineers? Are we all gluttons for punishment? But I repeat myself.

In particular: if there was a simpler language that could achieve the same goals, why wouldn't we prefer that? C, for instance, is ostensibly a much simpler language than Rust. Why does C's simplicity not yield the same results as Rust's complexity?

In this article, I'll attempt to make sense of all this. The best place to start is, in my opinion, the following question:

**What does complexity even mean, anyway?**


## Incidental complexity

Let us begin with a fairly easy task: Printing the product of two numbers. In Rust, this would look something like the following:

```rust
println!("{}", x*y);
```

Even this trivial example features some weird parts of Rust's syntax. There is the bang, which signifies a macro invocation. There's that `{}`, signifying arguments to be formatted within the string. And, depending on which languages you are used to, there's also the semicolon dangling at the end.

Wouldn't it be better if we didn't need all that? If we could just take a look at each part of the syntax and immediately identify what it's supposed to be doing?

Compare this with a language famed for its simplicity:

```brainfuck
[-]>[-]>[<+>-]<[ >>[<+<<+>>>-]<<<[>>>+<<<-]>-][-]>[-]>[<+>-]<[ >>[<+<<+>>>-]<<<[>>>+ <<<-]>-]>++++++++++<<[->+>-[>+>>]>[+[-<+>]>+>>]<<<<<<]>>[-]>>>++++++++++<[->-[>+>>]> [+[-<+>]>+>>]<<<<<]>[-]>>[>++++++[-<++++++++>]<.<<+>+>[-]]<[<[->-<]++++++[->++++++++<]>.[-]] <<++++++[-<++++++++>]<.[-]<<[-<+>]
```

Isn't this much better? Only 8 commands to remember. Each symbol does exactly one thing, and it's very easy to remember what each of them do. Down with syntax!

I'm not even going to touch on how ludicrously inefficiently Brainfuck uses the system's resources. I'm focused on one thing alone: What makes a language simple. And, indeed, Brainfuck is nothing if not simple! Many collections of programming challenges include the creation of a Brainfuck interpreter/compiler as a fairly easy exercise; the expected time is usually a few hours. So, is the latter program preferable to the former?

The answer is of course a resounding **no**, for one simple reason: while _each constituent part_ of Brainfuck's program is much simpler than Rust's, _the program as a whole_ is an explosion of complexity. Brainfuck is a Turing tarpit: Everything is possible, nothing is easy. Who cares that each command is easy to understand in isolation, when the program as a whole looks for all the world like chicken scribbles?

This snippet was chosen for its complexity, yes, but it serves to highlight a specific _kind_ of complexity. The thing we care about should work fairly simply, has no real reason to _not_ work fairly simply, but the language won't let us hide the implementation details behind a decent abstraction. This kind of complexity shall be referred to herein as **“incidental complexity”**, because there's no real reason for it.

A compiler will compile orders of magnitude more code than what comprises it: therefore, a more complex compiler that results in simpler code will lead to a huge reduction in complexity overall.

(Small rant: this is why I personally never understood the argument that “one of C's strong points is that you can write a standards-compliant compiler in a few thousand lines of code”. Well, Brainfuck takes one-hundredth that; why should I even care? Would such a compiler even be materially more useful than Brainfuck's?)


## Intrusive complexity

Having established that simplicity of programming is much more important than simplicity of implementation, we can proceed in our quest to seek a very simple language. Any self-respecting language permits abstraction of this sort, so we're thankfully spoilt for choice. For the purposes of this article, we shall select Python and Go as representative examples: Python is essentially executable pseudocode, and Go's entire design philosophy revolves around simplicity.

So, let's take a Go function that takes two channels as arguments. It then receives a value from the former and transmits it the latter:

```go
func tranceive (from chan int32, to chan int32) {
	val := <-from
	to <- val
	fmt.Println("Value transmitted: ", val)
}
```

Simple enough, right? Anyone could take a look at this function and figure out what it does, and what it prints.

And odds are most of them would be wrong.

See, the `a` and `b` arguments _might_ be useable, in which case _yes_, this value works as one would expect and it prints the value it transmitted. But the channels might also be uninitialised or closed, in which case this function could also deadlock, panic, or just read wrong data. (And that is, of course, assuming that `from` and `to` have working correspondents.)

In programming jargon, this sort of behaviour is known as a “footgun”. Footguns typically arise when a specific behaviour is very easy to _implement_, but very difficult to consequently _predict_.

There are more examples. Here's a very famous quote about the quintessential footgun, null references:

> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

Easy to implement. Difficult to predict the emerging behaviour. Sound familiar?

Let's take the other language mentioned above for its simplicity: Python. In Python, type hints are completely optional, and every operation just checks at run-time whether its arguments make sense. This means that generic code is very easy to write… but it also means that you can pass the wrong type of argument to a function arbitrarily deep in the call-stack. Best-case scenario, this leads to a panic; worst-case scenario, your data gets corrupted and you only realise much later. This also makes it much more difficult to connect the output of one library to the input of another! How do you know what data-types they return? Better hope the documentation is up to date…

C is much simpler than Rust. It also has approximately eleventy bajillion different ways to commit UB –Undefined Behaviour, that is– which is the programming language equivalent of a jury deciding that one slip of the tongue or one typographic error means that everything you declared becomes meaningless and can be interpreted in an arbitrary manner, and makes this decision without even informing you. Some times, you want this; most of the time, you _really_ don't.

All 3 of those languages –C, Python, Go– share the “maybe uninitialised, maybe closed” footgun we mentioned at the beginning. They also have the shallow-copy problem: The real-life equivalent would be if you asked another person to locate something in your notes, only for that person to scribble all over your notes without asking permission. “If I scribble on this piece of paper, will I end up overwriting my notes? Better copy that data just to make sure… wait, was _everything_ copied? Or just some of the things?”

These examples showcase a completely different example of complexity: not complexity in crafting a specific code-piece per se, but rather complexity arising from trying to combine different code-pieces together; not complexity in crafting _a_ path in the code, but complexity in dealing with _all possible_ paths in the code. This kind of complexity shall be referred to herein as **“intrusive complexity”**, because it sneaks up on you when you least expect it.

Software engineering is far from the only place in which intrusive complexity appears. Permit me a small detour to explain further.

### Intrusive complexity in manufacturing
Henry Ford is remembered to this day for making the most affordable cars of his era. What most people don't emphasise is that the components he used for his cars were expensive. The most crucial thing is this: his cars were not affordable _despite_ their components being expensive; they were affordable _because_ their components were expensive.

Ford cars were made in assembly lines. In them, each worker is unskilled, and does one thing: say, for instance, assemble part A and part B. But, if those parts are shoddily made, then they won't fit well with one another! They might need to be filed on one side, glued on another. This takes time and skill, and both of those things cost money. It follows that, for the assembly line to work as cheaply as possible, all parts need to be machined to precise tolerances, so they can be assembled together with minimal fuss. Interchangeable parts are expensive to _make_, yes; but they're so cheap to _assemble_ that their usage saves money on balance.

(Boeing used to know this lesson, but forgot it. The Boeing 777 was made out of very precise parts and was the most profitable aeroplane in Boeing's history; the Boeing 737 skimped out on its parts, and this cost Boeing very dearly.)

There's a reason for this, and it's called “combinatorial explosion”. If you create an assembly out of `N` parts, then the possible interactions between them are `N!`. For an assembly with 20 parts, that's 2432902008176640000 interactions. For 100 parts or more, we'd need entire lines of text just to write down the number.

Of course, not every part interacts with every other part. We can debate about the exact number of possible interactions between parts, as it fluctuates depending on system. The overall conclusion, however, ought to be a robust and universal law:

**Any complex system has many orders of magnitude more interactions than constituent parts.**

This also has an immediate corrolary:

**The possible complexity costs of part interactions are so overwhelmingly large, that eliminating them is worth any cost of complexity in each individual constituent part.**


## Inherent complexity

At this point, we're finally equipped to discuss the crux of this article: Inherent vs Emergent complexity.

All the languages we described above were inherently fairly simple. Any complexity that we ended up facing only came about when we actually tried to, er, actually do things with them. In other words, they had a small _inherent_ complexity, but large amounts of _incidental_ and _intrusive_ complexity—which we shall collectively refer to as “emergent”.

Rust takes the opposite approach. If anything, hiding inherent complexity from the user (possible error conditions, for instance) is an explicit _anti-goal_ of Rust. What Rust concerns itself with is the elimination of _emergent_ complexity, and its success in this is easily enough to explain the entire success of Rust as a language.

For instance: Yes, it occasionally gets very verbose having to clarify every trait and life-time your variables need. But the up-shot of this is that post-monomorphisation errors are eliminated. Even if the library functions you're using have no documentation, you can know everything you need to know about their correct usage through their signature alone—and that is a powerful, powerful thing.

A `uint32_t *array` is a pointer, and an `array: Option<&[u32; 2]>` is also a pointer; the former is simple, the latter is complicated. The latter's complexity, however, is _exactly what makes it simple to use:_ all the information you need to know is right there.

Unless a data-type explicitly supports interior mutability –like atomics do, for instance– then you can pass an immutable reference to _anything you want_ and you have a compiler guarantee that its contents will not change. No more getting your notes scribbled on when you lend them to someone!

The borrow checker is rightly considered to be one of Rust's more complicated subjects, no objections there. In fact, if all we care about heap-allocated data, all it does is complicate things! Yes, it saves memory, but that's not what this article is about: if you want simplicity, just copy everything. There's a reason why a common piece of advice for simplifying programming in Rust is “clone early, clone often”.

This, however, assumes that everything _can_ be copied—and that's only true inasmuch as heap-allocated data are concerned. As soon as we move beyond those, and get into things like handles for files or threads or servers, copying becomes a liability, not an asset.  Copying atomics renders them meaningless, and copying thread handles is a recipe for disaster. With that in mind, the borrow checker unlocks a _serious_ super-power: _Any reference to any of those things is compiler-guaranteed to find the underlying object in a useable state._ The amount of complexity this eliminates cannot be overstated. For instance, let's translate the channel example above into Rust:

```rust
fn tranceive (from: Receiver<i32>, to: Sender<i32>) {
    let value = from.recv().expect("Receiver appears to have hung up!");
    to.send(value).expect("Sender appears to have hung up!");
    println!("Value transmitted: {value}");
}
```

Like Go, there are possible error conditions. Unlike Go, both the possible error conditions and the way they are handled are transparent to the user. Most importantly, unlike Go, _none of those error conditions depend on the state of our arguments._ We may have no information about the correspondents of `from` and `to`, but `from` and `to` themselves are live by compiler guarantee. There's no possibility of deadlock, and there's no possibility of reading bogus data; at most, `from` might have to wait.

Take [Amos Wenger's comments on Go](https://fasterthanli.me/articles/i-want-off-mr-golangs-wild-ride): (My statements on Go are heavily based on his writings, BTW)
> [Go's simplicity is] a half-truth that conveniently covers up the fact that, when you make something simple, […] the complexity is swept under the rug. Hidden from view, but not solved.

He makes the same point, with different phrasings, over and over again [in his writings](https://fasterthanli.me/articles/lies-we-tell-ourselves-to-keep-using-golang).

Take [Bryan Cantrill's comments on C](https://bcantrill.dtrace.org/2020/10/11/rust-after-the-honeymoon/):
> The nothing that C provides reflects history more than minimalism; it is not an elegant nothing, but rather an ill-considered nothing that leaves those who build embedded systems building effectively everything themselves—and in a language that does little to help them write correct software.

Take [Jimmy Hartzell's comments on C++](https://www.thecodedmessage.com/posts/2022-05-11-programming---multiparadigm/):
> I’m not even sure I’d claim that C++ has too many features; it’s more that the features are not consistent. They clash with each other. Different feature-sets make assumptions that are violated by other feature-sets.

Take [Zachary Burns's comments](https://www.youtube.com/watch?v=4_Jg-rLDy-Y):
> The larger the program, the more chances there are for inconsistencies. Since any line could create an inconsistency with any other line, the potential for inconsistency grows exponentially as a function of the program's size. This is why large-scale development is difficult.

The blogs, writers, phrasings, and sometimes even topics can differ. But over and over and over, the same theme emerges: “Inherent complexity doesn't matter. What matters is emergent complexity.”


## Summary and conclusion

I believe that it is utterly pointless to try to categorise programming languages based on their complexity, unless one clarifies which type of complexity one refers to. I have chosen to categorise complexity in three groups –incidental, intrusive, inherent– as follows:

* **Incidental complexity** is the complexity that arises when something should be abstracted away, but isn't, thereby spilling its implementation details outside for no reason. This is important to mitigate, but also a solved problem since decades ago.
* **Intrusive complexity** is the complexity that arises when something important that should have been taken into account (say, an error condition) cannot be easily taken into account, plunging the program in an indeterminate state.
* **Inherent complexity** is the complexity that arises due to intrinsic specific properties of the tool we are trying to use, or the problem we are trying to solve.

So, which one of those matters the most? In the parlance of our times: Your mileage may vary. If you're writing throw-away code, where developer speed matters the most, maybe you'll prefer to sweep inherent complexity under the rug and whistle indifferently. Worse yet, if you're in a process of design iteration, ie if you need to keep rewriting and throwing away code again and again just to figure out what it is you're supposed to be building [like indie game devs do,](https://loglog.games/blog/leaving-rust-gamedev/), inherent complexity will be your biggest pain point by far. Steel is not always superior to [clay](http://www.thecodelesscode.com/case/118).

But if you want robustness, and consistency, and predictability, then you can't afford to ignore intrusive complexity. If anything, it's the enemy you'll want to eliminate first and foremost. And the language that eliminates it the most decisively and effectively is Rust.
