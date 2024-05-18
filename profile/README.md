# General 
Don't think of this as a regular ol' README file. Think of it like a happy child's Journal. This is an informative publication about the Landmark programming language and my the process and design choices while I was building it. Also, far below — after all of my chaotic thoughts — you'd be reading here, you would find documentation containing some snippets of Landmark code and others containing how to get started with using Landmark. After that description you might think of this as a whitepaper and that's wrong for three reasons:

1. Meh, you can argue that there's barely a problem I am trying to solve that another language which exists already does not solve. Although, I would talk about a few problems and the way Landmark tries to solve or sometimes even solves them.
2. Err... the pilot version of the compiler is written in Golang. No matter what reason sophisticated enough that I try to come up with, the tool would defeat this reason.
3. It's kind of weird calling it whitepaper when basically anybody reading this would read it in dark mode.

# And then it churned
I wanted to write a back-end framework in C but I quickly realized that it was a task that's too complicated for people. There were too many things that I needed to do myself like security, parallelization, extensibility, templating, and so on. The more code I fleshed out for this framework, the greater the amount of emptiness that was left in my soul. Then, in a beautiful halo of light, Rust descended from the skies and seemed perfectly suitable to fix all of the problems I was facing.

When I started playing around with Rust, and experimenting with stripped down versions of the framework I envisioned, I quickly realized that it was too complicated for people. While working out the parallelization pipeline in Rust, I felt like my head was leaving my body trying to find my way around lifetimes, and Thread programming in Rust. Much like the time I tried building the framework in C, you might argue it was a skill issue. Well... who knows?

At that time, I made the most intelligent decision and gave up on building the framework altogether. So, right now, why am I currently making the foolish decision of building this framework you might ask? While working with Rust, I fell in love with its syntax, memory management model (this is a partial truth), type system, and architecture. Then I thought, how can I borrow some concepts from Rust while making it more friendly for beginners? It was in that moment that I either conceived Landmark or I f**ked up — only time will tell.

# Designing Landmark
There were a lot of things I wanted to borrow from Rust but I had to pick my favorites so I stay in scope. The properties of Landmark that were inspired by Rust are its type system, memory management strategy, syntax, and how it handles abstraction (one more half-truth but we would talk about this in a bit). First, let's talk about the Landmark type system. In Landmark there are about 8 primitive type groups: boolean, numerical, text, tuples, objects, references, lifetimes, and functions.

## The `boolean` type
Unlike the previous paragraph might lead you to believe, there is only one boolean type in Landmark and as you might expect, it is named `boolean`. It can have only two possible values which are `true` and `false`. These two values are therefore called `boolean` literals. Like all real programming languages, boolean literals are not the only way to create boolean values. In Landmark, every logical expression evaluates to a boolean value. 

## The `numerical` types
In Landmark there are two families of numerical types: the [`integer`](https://www.freecodecamp.org/news/integer-definition/) types and the [`floating-point`](https://www.freecodecamp.org/news/floating-point-definition/) types. There are 9 `integer` types in Landmark which are `byte`, `int`, `uint`, `int16`, `uint16`, `int32`, `uint32`, `int64`, and `uint64`.

A `byte` is 8 bits wide. The `int` and `uint` types are one processor word wide, and this essentially means they'd be 64 bits wide on 64-bit machines and 32 bits wide on 32-bit machines. For the other integer types, their width is appending to their tail end. This means the `int16` and `uint16` types are 16 bits wide. In a like manner, any numerical type that is prefixed with the letter 'u' is [`unsigned`](https://www.thoughtco.com/definition-of-unsigned-958174).
    
In Landmark, there are only 2 number types floating-point numbers: `float32` and `float64`. A `float32` is a floating-point number that is 32 bits wide and in a like manner, a `float64` is a floating-point number that is 64 bits wide.

## The `text` types
In Landmark, there is only one real text type and that is a `string`. It is an `object` type which which means it provides one dimension of abstraction over the lower-level pitfalls typically associated with strings. Every usage of a string including string literals in Landmark, use the `string` type. This was done to simplify the developer experience. By the way, all strings in Landmark are encoded in UTF-8 by default. Strings in Landmark are composed using the `byte` type. A byte literal is a character literal and it is a valid character representation that is surrounded with the ' character.

There's another reference type that behaves like and can be used to compose a string which is a `ByteArray`. A `ByteArray` is a raw contiguous block of memory that represents a UTF-8 encoded string of `byte`s. It is typically always null-terminated whenever any API requires you to use it directly. Like they sound, these are C-type strings. You'd probably never have any reason to ever use them directly because they come with a foot-gun or two.

About how to declare string values in Landmark, there are 2 ways: using the \` character as the delimiter or using the " character as the delimiter. When the string is surrounded with the \` character, it is a raw string. This means that every white space character in the string would be printed out exactly how it looks. When the string is surrounded with the " character, every run of whitespace characters would be collapsed to a single space. Like this implies, all strings in Landmark are intrinsically multiline by behavior.

## The `tuple` type
In Landmark, there is a named type called a tuple. It is implemented like an anonymous structure at compile-time so it should be treated that way. A tuple must contain at least 2 entries of the same or different types. Otherwise, this would result in a compile-time error. The syntax used to declare and access entries in a tuple in Landmark looks something like:

```rust
let myTuple = tuple("Hello", 1, "Person");
let someElement: int = myTuple.$1; // Should be 1
```

After compiling Landmark to C, the variable in the example above `myTuple` might look something like this:

```c
struct tuple_xxxxxxxxx myTuple = (struct tuple_xxxxxxxxx) {
	.$0 = string__init("Hello", 5),
	.$1 = 1,
	.$2 = string__init("Person", 6)
};
int someElement = myTuple.$1;
```

## The `object` types
In Landmark, there are 3 "object" types and only one of them isn't an outright imposter. That is the any object that is an instance of a type declared as a `class`. In Landmark all such objects are essentially pointers. This means that in Landmark, the `structure` type is not a value type but a reference type. The reason this choice was made is so the developer would never have to worry about passing references to user-defined types to and from functions. Besides, memory cleanup becomes far less complicated when we know every user defined type is a reference type.

The second "object" type we have in Landmark is the `base` type. A base is similar to an `interface` in java. It serves no real purpose but to serve as the guideline for any classes that might want to take behavior from it. There are some `base`s in Landmark which carry specified behaviors that the compiler would treat as "primitive" to the language itself. Much like the interfaces in Java, Landmark bases can also be used to add bounds to a generic type. Exactly like an `interface` in Java, a `base` in Landmark can **only** contain functions. Yes, what Java calls interfaces, Rust calls traits — we move forward!

The third "object" type in Landmark is an `enum`. It represents any type in Landmark which contains a finite number of possible values which are simple enough that they can be represented as numbers (by the Landmark compiler). Much like in Rust, Landmark enums can also have instance functions. Unlike Rust however, the variant values in a Landmark enum do not contain any data.

## The `reference` types
In Landmark, there are 

<!--
# Getting Started with Landmark
<aside name="home">
If you are reading this, I might want to assume you technically do not have the development environment. 

If that is truly the case, you can download the development environment from [here](https://github.com/landmark-lang/.github).

</aside>
-->
