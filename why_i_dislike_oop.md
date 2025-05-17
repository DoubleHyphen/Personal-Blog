# Traditionally-implemented Object-Oriented Programming is unfit for purpose

## Introduction

The title of this post makes a bold claim, I understand. It has been specifically phrased as to avoid ambiguity—still, it needs some clarifications and caveats.

1. Not everyone agrees 100% on what Object-Oriented Programming (henceforth “OOP”) constitutes. By Smalltalk's standards, neither C++ nor Java are OOP; by my own, Rust is OOP. I chose the phrasing “traditionally-implemented OOP” to refer to the most popular meaning, which includes inheritance; this is exemplified eg by C++ and Java. (Henceforth “tradOOP” for short.)
2. The purpose of inheritance, in each and every explanation I could find, is mentioned to be the structuring of _is-a relationships_, explained further below. This is the purpose for which I believe inheritance to be unfit, and by extension tradOOP as a whole. I do not doubt the existence of programs for which it is a good fit, or even the best we currently have; my assertion concerns itself with its stated purpose, and only with that.
3. The main observation presented herein is not mine; it even exists in Wikipedia, with the name “circle-ellipse problem”. Wikipedia's article, however, is not accessible to the casual reader, and does not offer alternatives. The present article, in contrast, aims to achieve both those goals.


## Our motivating question

Let us begin with a simple enough question, whose answer will stay with us throughout the rest of this article: **What purpose does a programming language serve?**

For the purposes of this article, the working answer to this question shall be as follows:

> **The purpose of a programming language is to map real-world problems to a computer's executive capabilities.**

Correct, isn't it? We could go more specific or more vague if we wanted, but this answer is not wrong at all. More to the point, this answer leads us to a very important corollary: 

> _One direct measure of a programming language's quality is its ability to effectively model the real world._

I say this, because there is a very specific part of the real world which I would like to high-light, and explore its relationship with programming languages: _subcategories_, and its very close relative _substitutability_.

### Is-a relationships
Subcategories appear in natural language fairly frequently. Usually, they appear using the phrase “is a”, hence the term “is-a relationship”. For instance: A woman _is a_ human. A human _is a_ mammal. A mammal _is an_ animal.

Substitutibility means that, whenever one wants a Y, you can instead offer an X without any problems; in other words, you _substitute_ an X for a Y. The reason why it's a very close relative of subcategories is because, if X is a subcategory of Y, then an X can be substituted for a Y. For example, thanks to the hierarchy presented in the prior paragraph, if someone says “Name an example of a specific animal”, a completely correct thing to respond would be “your mother”.

(Sorry about this. Won't do it again.)

With all this in mind, let us examine how subcategories and substitutability (henceforth “subtyping”) work in Rust, and compare/contrast it with object-oriented languages like C++ and Java.


## Subtyping in Rust

To illustrate Rust's capabilities in subtyping, we will be using a simple hierarchy of geometric shapes. We shall structure it as follows:

* A parallelogram will be represented by three points: Two of its consecutive vertices, plus its centre.
* A parallelogram's angle or side can be changed at will. We can use this to create new ones if we so desire.
* A rectangle is a parallelogram whose angles are all equal.
* A rhombus is a parallelogram whose sides are all equal.
* A square is a rhombus and rectangle at the same time.

The phrasings were not really chosen at random. They were chosen to high-light a very specific way to define things: So-called _genus and differentia._ In simpler terms, you define something by saying first which category it belongs in, and then what sets it apart from other members of this category.

So, let's explore how this whole thing could be expressed in Rust! The first thing to note is that, if X and Y are both nouns, Rust has no direct way to express the phrase “an X is a Y”. Instead, we have the following two options, each with its own unique advantages and disadvantages:

1. One could choose, for the Y, not a noun but an adjective: in Rust terms, not a concrete type but a trait. Big advantage: We can identify common behaviour between types, thereby grouping types together by the behaviour they exhibit. Big disadvantage: We can no longer say “let there be a Y called `foo`”, because traits cannot be used to instantiate variables.
2. The other option, if Y is a noun, is to implement the `From` trait. This is a way to tell the language how to _transform_ an X _into_ a Y, which in turn lets us use it in place of a Y. Big advantage: We can have concrete instances of both Xs and Ys. Big disadvantage: Once we change the type of something, its previous type gets lost, and thenceforth we can _only_ use it as a Y.

Of course, nothing prevents us from using both of those approaches at the same time! We do exactly this in the code below: There exists a `Parallelogram` data-type, which represents a generic parallelogram; there also exists an `IsPgram` trait, which represents the behaviours common to all of subcategories of parallelograms.

The code is as follows. The most important points follow immediately after.

```rust
// Not pertinent to the current discussion.
#![allow(refining_impl_trait)]


type Length = f32;
type Angle = f32;
type Point = nalgebra::Point2::<Length>;

pub struct Parallelogram {
    primary_point: Point,
    centre: Point,
    secondary_point: Point,
}

pub struct Rectangle {
    // NOTE: To ensure uniqueness, all points and 
    // sides will be examined clock-wise.
    primary_point: Point,
    centre: Point,
    primary_side: Length,
}

pub struct Rhombus {
    // NOTE: To ensure uniqueness, all points and 
    // sides will be examined clock-wise.
    primary_point: Point,
    centre: Point,
    primary_side: Length,
}

pub struct Square {
    primary_point: Point,
    centre: Point,
}

trait IsPgram: Into::<Parallelogram> {
    fn set_primary_angle(self, angle: Angle) -> impl IsPgram;
    fn set_primary_side(self, length: Length) -> impl IsPgram;
}

impl IsPgram for Parallelogram {
    fn set_primary_angle(self, angle: Angle) -> Self {...}
    fn set_primary_side(self, length: Length) -> Self {...}
}

impl Parallelogram {
    fn set_primary_angle_in_place(&mut self, angle: Angle) {
        self = self.set_primary_angle(angle);
    }
    fn set_primary_side_in_place(&mut self, length: Length) {
        self = self.set_primary_side(length);
    }
}

impl IsPgram for Rectangle {
    fn set_primary_angle(self, angle: Angle) -> Parallelogram {
        self.into::<Parallelogram>().set_primary_angle(angle)
    }
    fn set_primary_side(self, length: Length) -> Self {...}
}

impl Rectangle {
    fn set_primary_side_in_place(&mut self, length: Length) {
        self = self.set_primary_side(length);
    }
}

impl IsPgram for Rhombus {
    fn set_primary_angle(self, angle: Angle) -> Self {...}
    fn set_primary_side(self, length: Length) -> Parallelogram {
        self.into::<Parallelogram>().set_primary_side(length)
    }
}

impl Rhombus {
    fn set_primary_angle_in_place(&mut self, angle: Angle) {
        self = self.set_primary_angle(angle);
    }
}

impl IsPgram for Square {
    fn set_primary_angle(self, angle: Angle) -> Rhombus {
        self.into::<Rhombus>().set_primary_angle(angle)
    }
    fn set_primary_side(self, length: Length) -> Rectangle {
        self.into::<Rectangle>().set_primary_side(length)
    }
}

impl From<Rhombus> for Parallelogram {...}
impl From<Rectangle> for Parallelogram {...}
impl From<Square> for Parallelogram {...}
impl From<Square> for Rectangle {...}
impl From<Square> for Rhombus {...}
```

There's no need to read all this line by line. The important points here are as follows:
1. As mentioned prior, parallelograms are represented both as categories (`IsPgram`) and as concrete types (`Parallelogram`).
2. Each data-type is represented using exactly the state it needs in order to be uniquely represented, no more and no less. Parallelograms need 6 numbers, rectangles and rhombuses need 5, and squares need 4.
3. Each member function can, and does, denote the invariants it does and does not maintain. A rhombus can change its primary angle without changing its data-type; not so a rectangle. This goes vice-versa for changing the primary side.
4. Some member functions can be implemented by changing the data-type, then deferring to an already-existing implementation.
5. Common behaviour is described in the trait, unique behaviour is described separately in each concrete data-type.
6. Not _all_ possible states here are valid. Sides can have upper or lower limits, lengths must be positive, floats must be finite. Those will have to be maintained using sanity checks in the constructor. That said, the degrees of freedom are exactly the ones we want.

This is more-or-less it. Let's examine the merits and demerits.

Merits:
* Very accurate modelling of the problem
* State is never wasted

Demerits:
* Highly unintuitive to write
* The differentiæ are not explicit in the code
* The concept of “parallelogram” is modelled twice in the code, with subtle differences
* Fairly verbose and repetitive (just the `From` implementations take a fair bit of code)

Ho hum. Two merits, four demerits. Not horrible, but definitely not great.

Can tradOOP do better?


## Subcategories in tradOOP

Subcategories in tradOOP are so tightly connected with inheritance as to be basically synonymous. Briefly: when an X is declared to be a Y, it inherits all its state and behaviour. After that, it can expand either or both of those things as it sees fit.

Let's translate the above example to old-style C++, as a representative example. I am purposefully avoiding modern C++ idioms, because as mentioned above I am aiming to write about tradOOP specifically.
```cpp
class Point {
    float x;
    float y;
};
class Parallelogram {
    private:
        Point anchor_point;
        Point centre;
        Point primary_point;
    public:
        Parallelogram set_primary_angle (float prim_ang) {...}
        Parallelogram set_primary_side (float prim_sid) {...}
        void set_primary_angle_in_place (float prim_ang) {...}
        void set_primary_side_in_place (float prim_sid) {...}
};

```
Much tidier, isn't it? Unlike Rust, we have no need to separate `Parallelogram` into two; the same class models both the state and the relationships. Convenient!

On to rectangles:

```cpp
class Rectangle: Parallelogram {
    // Uhhhhhhhhhhhhhhhh…
};
```
…uh-oh.

Just like that, we encounter the first insurmountable problem.

### Inheritance bestows state
Said problem is as follows: If we denote `Rectangle` to be a subcategory of `Parallelogram` via inheritance, we automatically bestow to it all the state that the latter already has. This, however, goes directly against the entire concept of the _genus and differentia_, which we mentioned earlier.

Think of it this way: the entire job of the differentia is to take a genus and constrain its possible members. Thus, if the possible members have been constrained, their representation ought to need _strictly less_ state than the representation of the genus. Inheritance disagrees, and decides that their representation will need _weakly more_ state than the representation of the genus. Which is, even in the absolute best case, a complete waste of resources.

If we want to constrain the possible states, we have to go the other way, and denote `Parallelogram` to be a subcategory of `Rectangle`. This solves some problems but creates 10× as many, because now we can use any random `Parallelogram` wherever a `Rectangle` is expected.

But fine, whatever. Let's say we don't care about wasting state. Let's keep going and see how to implement the methods:

```cpp
class Rectangle: Parallelogram {
    private:
    // Nothing here, we already have more state than we need.
    public:
        Parallelogram set_primary_angle (float prim_ang) {...}
        Rectangle set_primary_side (float prim_sid) {...}
        void set_primary_side_in_place (float prim_sid) {...}
        void set_primary_angle_in_place (float prim_ang) {
            // SUNNUVA---
        }
};
```

And just like that, we encounter the _second_ insurmountable problem.

### Behaviour can never be constrained
In the Rust example earlier, we had both `in_place` versions of the methods (which merely modified an already-existing variable) and ordinary methods, which created a new copy. The important thing to note is this: The `in_place` methods _only existed for specific data-types,_ because they're not common to all of them!

The C++ example above has no way to declare this. As soon as we create a subcategory of a `Parallelogram`, it _has to_ have at least the same behaviour as the `Parallelogram`, including the `in_place` methods. With the `side` one there is no problem, but the `angle` one _has to_ make a `Rectangle` change its angle, in-place, while remaining a `Rectangle`. Which is the opposite of what we want.

There is a third problem, more minor but still noteworthy.

### Composition _also_ bestows state
A common piece of advice for tradOOP languages is to favour “Composition over inheritance”. Briefly put, this says that when we want to ensure that `X` has at least as much state as `Y`, we ought to do that by just including a `Y` as a member of each `X`, not by writing `class X: Y`.

This is, in my humble opinion, a disappointing duplication of concerns. Two data-types with the same _behaviour_ have no reason to have the same _state_; indeed, as we showed earlier, the most natural way to describe things is the exact opposite. We are told to favour one over the other, when _they should never have stepped on each other's toes in the first place._


## Summary and conclusion

The most natural way to think of a subcategory is as something that has the following two properties:
1. It can be used (“substituted”) wherever its super-category can
2. It needs strictly less state to be described, compared to its super-category.
3. It has some behaviour in common with its super-category, though not necessarily all of it.

Traditional OOP, by using inheritance to achieve the former part, automatically defenestrates the other two. It is exactly this weakness of tradOOP which IMHO makes it lose to Rust in its own turf. And to think: Rust does not really do a great job at modelling those properties either! But, unlike tradOOP, at the very least it can model them all at the same time.


## Postscript

Credit where it's due: I think method-calling syntax is great, and am grateful to tradOOP for introducing it to the world.

I've seen someone denigrate OOP in general, saying among other things that method-calling syntax can't be all that significant or useful. “Have you seen headlines writing ‘Method-calling syntax introduced, giving developers world-wide huge boosts in productivity’?”

This seems to me to be an example of motivated reasoning: beginning from the axiom that OOP is bad, we look for excuses to support the idea. I disagree, and have a very strong example in mind: `point_1.route_to(point_2)` is visibly different that `point_2.route_to(point_1)`, which in turn is visibly the same as `point_1.route_from(point_2)`. Method-calling syntax lets us distinguish between the subject and object of a phrase, making some things much clearer. Without this syntactic sugar we'd need to write `route(point_1, point_2)` which makes it extremely easy to misuse, because we don't have immediate hints as to which point is the beginning and which is the destination. There's also the possibility of `route_from_to(point_1, point_2)` but it doesn't really solve the problem IMHO.

And that's before we get into the fact that method-calling can be chained, leading to an operation being read very naturally left-to-right: `point_1.route_to(point_2).via(point_3).using(mode_of_transport)`. Without method syntax this would be written as `using(via(route(point_1, point_2), point_3), mode_of_transport)`. This alternative, instead of being read left-to-right, needs to be read in a spiral manner from the middle towards either end. I think the readability does not even compare.
