# Unison

What's interesting about Unison up front is the marriage of Smalltalk, Erlang, Forth and Lisp runtime style, with git or blockchain-like version control, with statically typed functional programming. Unison tags itself the friendly programming language from the future, and I can see why, plus a number of challenges that could make it hard to adopt.

"Erlang allows two versions of any given module in a library to be active at the same time." - http://christophermeiklejohn.com/riak/erlang/2019/08/29/versioning.html

Unison goes further than that - it refers to functions by a hash of their implementation, similarly to how Git repositories contain previous versions of hunks, and the names are more of a convenience for us humans. It has no limits, there can be 100 versions of the same function floating around, all with the same name, or there could be 100 names for one implementation. The types Maybe and Optional, are just separate names for the same data structure.

That's all well and good for machines, and can be used to great effect in code review - I'll pull your changes in, but in a different module so I can review them, test them out alongside the original implementations - but those pesky hoo-mans won't thrive so well if the old names stick around forever, so Unison does encourage you to resolve things back down to 1 version. Along the way, all your code will remain compilable, so you can try things out and keep running, much like the [Erlang 1980s style telephone exchange video](https://www.youtube.com/watch?v=xrIjfIjssLE).

```
The namespace has 1 transitive dependent(s) left to upgrade.
Your edit frontier is the dependents of these definitions:

  unique type .myWork.Box

I recommend working on them in the following order:

1. toText : myWork.Box -> Text
```

## Syntax

Unison looks a lot more like Haskell than any other language, and its implementation language is Haskell. So typically instead of using `"foo".length()` as you might somewhere with OO influence such as Java, Kotlin, Dart, Rust, you'll use `length "foo"`. I think I would actually miss the dot syntax, and wonder if, as Unison is not too tied to the actual code you write, but stores everything as an AST, it might allow some dot syntax in a separate grammar. Designing that would be interesting. Rust seems to do a good job of supporting the OO dot syntax, without actually caring about being an OO language, by having the developer declare that a parameter is Self. I could see that being inferred by Unison, or declared through imports. Let's explore that, just for fun:

### Today's Unison

```
use List.map
map (x -> x * 10) [1,2,3,4,5,6]
```

### Oonison (please don't actually use that!)

```
use List.map
[1,2,3,4,5,6].map(x -> x * 10) -- assumes that the last parameter is Self, opposite to Rust, for partial application to work well by default.
```

Or, if we can annotate List.map itself to say which is Self:

```
List.map : (a ->{𝕖} b) -> Self@[a] ->{𝕖} [b]
```

Anyway, that's just a thought, and maybe there's a decent way already of chaining calls similar to dot-using languages without having to read back to front.

I also wonder how far away this language from the future is from not needing to define a textual representation at all - I don't see myself dragging blocks around a la Scratch all day long, but maybe we're a hop or two away from never having to worry about trivialities like parameter order (Self or otherwise), because the AST stores them as an unordered set instead of a list.

An area that surprised me is that there are no [operator precedence rules](https://www.unison-lang.org/learn/language-reference/syntactic-precedence-operators-prefix-function-application/). I've worked with that before, e.g., Lisp, where 2 + 3 * 4 - 1 needs to be written `(- (+ 2 (* 3 4)) 1)`. Unison is a little better, we can treat it like a child who hasn't learned [BODMAS](https://en.wikipedia.org/wiki/Order_of_operations) yet: `2 + (3 * 4) - 1`. James Ward (I think) on his [Happy Path Programming](https://happypathprogramming.com/) podcast recently described Lisp as being written more for machines than for humans, kind of ironic given the SICP quote about code being only incidentally for machines, but I do see James' point. This lack of precedence will be hard to change later, even with automation, as by then there may be incorrect code around that assumed Unison did precedence 'correctly'.

The Unison documentation reads: "All operators and infix function applications currently have the same precedence", so I hope for the sake of the project that the word 'currently' implies this can still change.

## Tooling

Everything about Unison is custom - it doesn't gel well with existing version control systems. There is the Unison Code Manager that resembles and embeds a REPL, and embeds flows like reviewing pull requests, pulling from upstream repositories. The ideas here are great, and in a Unison-only environment this would work well, but I wonder if there would be a benefit to still being able to see the codebase as a bunch of text files, even if some of them contain prior versions of functions and types. I've seen Rust's Cargo being a potential problem for adoption of that language - as Rust is built around Cargo then if you're going to use another build tool instead, you can expect some level of pain. Unison is even more tied to its toolchain; that would likely need some automation to extract text, to allow corporate code checkers to find, say, stray DB passwords checked in.
