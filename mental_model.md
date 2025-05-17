# Programmers' mental models

## Introduction

What does a programmer think about while programming?

The question might sound simple: “About the program, duh!” is a possible glib, yet very accurate, response. But it is completely insufficient for our purposes. After all, a program is primarily a means of communication: its entire purpose is to bridge, as well as possible, the gap between what humans can express and what computers can understand. It must be understandable by the programmer's current as well as future self, possible collaborators or colleagues, and of course by the machine. All software has a recipient—if it did not, we wouldn't need software, just a blackboard and chalk.

In this article I'll present a few quotes I've read, which initially made absolutely no sense to me. There does exist a theory, however, that handily explains all of them, and ties into the question we posed at the beginning.


## _Smart and gets things done_

Quote the first comes from Joel Spolsky, famous writer on software. In his book on hiring employees, _Smart and gets things done,_ he mentions the following:

> I’ve come to realize that understanding pointers in C is not a skill, it’s an aptitude. In first-year computer science classes, there are always about 200 kids at the beginning of the semester, all of whom wrote complex adventure games in BASIC for their PCs when they were 4-years old. They are having a good ol’ time learning C or Pascal in college, until one day their professor introduces pointers, and suddenly, they don’t get it. They just don’t understand anything anymore. […] _For some reason most people seem to be born without the part of the brain that understands pointers._

Cue confusion on my part. Surely pointers can't be all that difficult, really?

I initially ignored this. Maybe there's just something special about pointers.


## _Maybe Not_

Quote the second comes from Rich Hickey, creator of Clojure. In a talk of his, titled _Maybe Not,_ he mentions the following:

> Maps are the most fundamental functions in programming. They should not be denigrated. They should be exalted. This is the first place to start. This is the simplest thing that you can do. There is no code associated with it!

(Note: By “Maps” here, he refers to what's called `std::map` in C++, or `HashMap`/`BTreeMap` in Rust)

At this point, I disagreed so strongly with the speaker that I closed the tab. Maps, the most fundamental functions‽ The most fundamental function is `NAND`, because it's quick to implement in silicon and can produce everything else. “No code associated with it”‽ You can't even use them without a memory allocator!

Whatever, let's leave this person to his own devices. He's a celebrity already, clearly he must be doing _something_ right.


## “I'm sorry, you used _binary operations‽_”

Quote the third comes from a web-site which I have sadly since lost. There were a few pieces of advice towards programmers, and one was to write readable code—a fairly obvious one. But the example presented as unreadable was the following:

```c
next_step = prev_step >= 0 ? prev_step << 2 : (-prev_step) << 2;
```

which I didn't think was all that bad, really. In any event, the readable alternative proposed was 

```c
next_step = Math.abs(prev_step) * 4;
```

which… OK, completely fair. But then the writer warned: "Don't use unreadable things such as binary operations!”

I could only scoff at this. What sort of remotely competent programmer doesn't know what binary operations are? They're even more fundamental than addition, for crying out loud. If you thought this was unreadable, wait until you see branchless GPU programming[^¹]!

[^¹]: Just for fun, the equivalent branchless GPU code would be something like 
```c
uint32_t mask = 0 - (prev_step >> 31);
next_step = ((prev_step ^ mask) - mask) << 2;
```
assuming that `prev_step` is a 32-bit integer. How's _that_ for unreadable?


## The moment of epiphany

The fourth quote, and most crucial one, came from Wikipedia's article on systems programming. It was very short, just 9 words:

> Systems programming requires a great degree of hardware awareness.

And as soon as I read it, my breath hitched, and the epiphany dawned on me.

See, if there's such a thing as “hardware awareness”, there must correspondingly be such a thing as “hardware _unawareness_”. And if there's such a thing as hardware unawareness, then _every single weird thing mentioned earlier immediately makes sense,_ and can be explained by programmers who think of mathematics instead of hardware. To wit:

* Pointers are how computers manage to remember what's been stored where. They don't exist in mathematics—if there were no computers, there would be no pointers.
* Maps _in programming_ are complex pieces of software. Maps _in mathematics_ are the most general, primitive, and simple way to make outputs correspond to inputs. 
* Binary operations only exist because they're dirt-cheap to implement in hardware. If hardware was not a concern, binary operations would not be a concern either.


## The two mental models

With all of the above in mind, I think we can broadly categorise programmers in two categories, depending on their mental model, as follows:

* _Category A_ thinks of a specific machine, with a specific instruction set, memory model, etc.
* _Category B_ thinks of a black box which is really good at doing mathematics.

Which is better is, of course, more of a matter of circumstance than anything else. An embedded programmer who does not have a clear mental image of the controller being programmed won't remain an embedded programmer for long. On the other hand, a web developer has no way to know in advance in which devices a program will run! If in addition most operations are bottlenecked by network or I/O, then hardware unawareness might even be a good thing to have.
