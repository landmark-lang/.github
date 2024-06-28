# General 
Don't think of this as a regular ol' README file. Think of it like a happy child's Journal. This is an informative publication about the Landmark programming language and my the process and design choices while I was building it. Also, far below — after all of my chaotic thoughts — you'd be reading here, you would find documentation containing some snippets of Landmark code and others containing how to get started with using Landmark. After that description you might think of this as a whitepaper and that's wrong for three reasons:

1. Meh, you can argue that there's barely a problem I am trying to solve that another language which exists already does not solve. Although, I would talk about a few problems and the way Landmark tries to solve, or sometimes even solves, them.
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

A `byte` is 8 bits wide. The `int` and `uint` types are one processor word wide, and this essentially means they'd be 64 bits wide on 64-bit machines and 32 bits wide on 32-bit machines. For the other integer types, their width is appended to their tail end. This means the `int16` and `uint16` types are 16 bits wide. In a like manner, any numerical type that is prefixed with the letter 'u' is [`unsigned`](https://www.thoughtco.com/definition-of-unsigned-958174).
    
In Landmark, there are only 2 number types floating-point numbers: `float32` and `float64`. A `float32` is a floating-point number that is 32 bits wide and in a like manner, a `float64` is a floating-point number that is 64 bits wide.

## The `text` types
In Landmark, there is only one real text type and that is a `string`. It is an `object` type which which means it provides one dimension of abstraction over the lower-level pitfalls typically associated with strings. Every usage of a string including string literals in Landmark, use the `string` type. This was done to simplify the developer experience. By the way, all strings in Landmark are encoded in UTF-8 by default. Strings in Landmark are composed using the `byte` type. A byte literal is a character literal and it is a valid character representation that is surrounded with the ' character.

There's another reference type that behaves like and can be used to compose a string which is a `ByteArray`. A `ByteArray` is a raw contiguous block of memory that represents a UTF-8 encoded string of `byte`s. It is typically always null-terminated whenever any API requires you to use it directly. Like they sound, these are C-type strings. You'd probably never have any reason to ever use them directly because they come with a foot-gun or two and the overhead of actually having to count the number of characters in the string.

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
In Landmark, there are 3 reference types: the `any` type, the `ByteArray` type, and the `Ref<type T>` type. We've already talked about the `ByteArray` type before so we would not be covering it here anymore. For that reason, let's look at the `any` type first.

Unlike the very misleading name, the `any` type actually cannot contain a value or memory of <span style="color: #f02020">__any__</span> type. It just represents a pointer to a chunk of memory. In fact, undef the hood, it is directly just the C `void*` type. To make working with them easier in Landmark, the only types that can casted to and from this type are other reference types.

This means that to cast to and from the `any` type, you need to obtain the reference of the value first. To do that in Landmark, you would use the `Ref` class which brings us to the next reference type. The `Ref<type T>` type is really just a pointer. It is only a handy way of dealing with pointers with one dimension of abstraction. The `Ref<T>` type was created to make working with pointers a lot easier and for that reason, there are some rules that this type imposes for its use:

1. You can never build a `Ref` to another reference type. This means that not only is something like a `Ref<Ref<int>>` banned but something like `Ref<any>` or `Ref<ByteArray>` is banned too.
2. If a `Ref` is no longer valid and you attempt to access it, it would throw an error that CANNOT be "caught."
3. To obtain a reference to a value type (numerical, boolean, tuple, or enum types), you MUST use the `Ref::copy` function. You cannot obtain a reference to a value of any of the highlighted types with the `Ref::to` function.
4. To obtain the reference of a class object, you MUST use the `Ref::of` function. You cannot obtain a value to a class object using the `Ref::copy` function. If you want to duplicate a reference to a class object, you use the `Ref.clone` function.
5. You cannot obtain the reference to a lifetime, function or variable that has a functional type.
6. When working with references of value types, much different from class objects, direct mutation is not possible. To store any changed values inside of the reference, you must use the `Ref.update` function.

Let's attach a simple example of how to use the reference type and the different situations you encounter it in below.

```rust
fn moveInto(arg: mut Ref<string>) {
	arg.update("Hello World");
}

fn moveInto(arg: mut Ref<int32>) {
	arg.update(1337);
}

let mut stringRef = Ref<string>::from("Initial value");
let mut intRef = Ref<int>::from(0);
let string1 = stringRef.deref();
let int1 = intRef.deref();

// Now, mutate this reference.
moveInto(mut stringRef);
moveInto(mut intRef);

// Next, do something strange.
let string2 = stringRef.deref();
let int2 = intRef.deref();

// If these two strings do not have an equal value...
if string1 != string2 {
	panic("The two strings should have identical values at this point in time.");
}

// If these two are equal values, then everything has gone to s**t.
if int1 == int2 {
	panic("This makes no sense because the two numbers should have different values right now.");
}
```

As you would notice from the code above, the behavior of a references to a class object is identical to the behavior of pointers in C or any other systems programming language. However with the reference to a value type, the moment you dereference it, it loses any connection to the reference it originated from.

## The `Lifetime` type
I know I said Landmark is heavily inspired by Rust in most of its semantics and that's true for the most bit. I'd try to talk about lifetimes in Landmark without getting too much into the details of how memory is managed in Landmark. In Landmark, lifetimes are treated like a '__value__' and not information.

This means that it can be passed around as an argument between functions and required as parameters for functions. On this note, the standard library functions for allocating memory in Landmark require a lifetime as an argument. In Landmark, lifetimes can have 3 possible literal values: `__static`, `__object`, and `__local`.

The `__static` lifetime is equivalent to the `'static` lifetime annotation in Rust. Any memory allocated with this lifetime would last for the entire runtime of the program and would only be freed when the program is terminated.

The `__local` lifetime is only valid within a function and represents the lifetime associated with a single invocation of the function. It does not carry over between invocations of the same function. This lifetime is deallocated right before the function returns to its caller.

The `__object` lifetime is only valid within an instance class method. It represents the lifetime of the `this` object or the "__current instance__" of the class you are working on. When the object is deallocated, any object that was bound to its lifetime would be deallocated too.

When a `Lifetime` is due to get deallocated, we say it has gone out of scope. When a `Lifetime` gets deallocated, all objects and chunks of memory allocated in reference to it would be deleted. No reference counting included. For this reason, there are functions in the standard library that allow you move a reference to another lifetime.

## The `functional` types
In Landmark, functions can also be treated as a value which gave reason to functional types. To declare a variable with a functional type in Landmark, you use the syntax below:

```rust
let fnValue: fn(string, int32) -> string;
```

Following what you see above, you can split this into two parts:
1. The parametric information of the functional type.
2. The return type information of the functional type.

The parametric information is what gives the compiler information about the inputs of the function. In the example above, the parametric information is `fn(string, int32)` which means that this is a function that __consumes__ a string, and a 32-bit integer.

The return type information is what gives the compiler information about the output of a function. In the example above, the return type information is `-> string`. It can be interpreted loosely as "gives a string as output." When the return type information is omitted in a functional type, it means that the value function DOES not return a value. In C terms, that means the return type is `void`.

To create a value function in Landmark, you use the syntax shown below:

```rs
let myFn = fn(foundation: string, number: int32): string {
	return foundation + number;
}
```

It is worth noting that in Landmark, there is no such thing as a closure capturing any values from the scope outside of it. At least, for now. Maybe I might include one in the future, who knows?

# Managing memory in Landmark
In Landmark, memory is logically grouped into lifetimes. Now, we've talked about most of this when we were analyzing the `Lifetime` type so I would avoid repeating some of the things I already said there before. To think about memory in Landmark, we must first think of a program as a collection of two types of procedures:

1. Procedures that yield results.
2. Procedures that yield no results.


## Procedures that yield no results
These are functions that have no return type annotated on them. They are compiled to have the `void` return type in C. There are no magic lifetime parameters in such functions. We can therefore say that the lifetimes of these functions is self-contained within the functions.

For this class of functions, the lifetime of any piece of memory allocated within them would be `__local` to that given invocation. Meaning all class objects, and references that were generated inside of this function would be cleaned up right before this function returns to its caller.

To give an example below, let's consider a function that allocates a new object of type `Map<string, string>` from the heap and adds a single entry to it then returns to its caller.

```rs
fn useMap() {
	let mapRef: Ref<Map<string, string>> = new<Map<string, string>>(__local);
	let map = mapRef.deref();

	// Now do something to the map.
	map["message"] = "Hello World!";
}
```

Would be something similar to the code you have below after being compiled to C

```c
void useMap() {
	struct Lifetime* __local = $___lmk_lifetime_new();
	struct type_xxxx* mapRef = new$xxxx(__local);
	struct type_xxxx* map = fn_xxxx_deref(mapRef);

	// Now do something to the map.
	fn_xxxx_set(map, $__lmk_make_str(__local, "message", 7), $__lmk_make_str(__local, "Hello World!", 12));

	// Now, destroy this lifetime.
	$___lmk_lifetime_destroy(__local);
}
```

## Procedures that yield results
These are functions that have an annotated return type. There is a magical lifetime parameter bound to such functions. This parameter is where the lifetime of the topmost function scope from this procedure that yields a result.

Let's examine some Landmark code below to attempt to illustrate this example below:

```rs
fn makeMap(): Map<string, string> {
	return new<Map<string, string>>(__local).deref();
}


fn useMapToChange(): boolean {
	// This is fine.
	let mut map = makeMap();

	// Fair enough.
	map["getter"] = "Getter";

	// Either this one, or that one.
	return true;
}

fn doItAll() {
	// If this is not the case...
	if !useMapToChange() {
		panic("The universe has imploded.");
	}
}

fn main(env: mut Environment) {
	doItAll(); // Just call this directly.
}
```


This would transform to code that looks more like this in C:

```c
struct type_xxxx* makeMap(struct Lifetime* __local) {
	return fn_xxxx_deref(new$xxxx(__local));
}

bool useMapToChange(struct Lifetime* __local) {
	// This is fine.
	struct type_xxxx* map = makeMap(__local);

	// Set this here.
	fn_xxxx_set(map, $__lmk_make_str(__local, "getter", 6), $__lmk_make_str(__local, "Getter", 6));

	// Then do this.
	return 1;
}

void doItAll() {
	// This is fine.
	struct Lifetime* __local = $___lmk_lifetime_new();

	// This is fair enough.
	if(!useMapToChange(__local)) {
		panic($__lmk_make_str(__local, "The universe has imploded", 25));
	}

	// Destroy this scope.
	$___lmk_lifetime_destroy(__local);
}

void main(struct type_yyyy* env) {
	// This is also fine.
	struct Lifetime* __local = $___lmk_lifetime_new();

	// Then call the function.
	doItAll();

	// This is fine.
	$___lmk_lifetime_destroy(__local);
}
```

If you could read and understand what was going on in the C version of the code, you'd quickly notice that the `Map<string, string>` that is created in the `makeMap` function does not get deallocated until the function `doItAll` returns to its caller — `main`. This is exactly the kind of behavior we were talking about.

The one that makes sure that the moment it is inconceivable for any piece of memory to still be referenced by any code, we drop it. This way memory is only freed at the point where we are sure it no longer has any reason to be allocated.

## There's a problem though
Yes, there is a problem though that this method of managing memory introduces and I acknowledge it. The moment you, by unfortunate coincidence, have a chain of functions that return a result to their caller and that chain goes all the way up to main, you effectively never free any memory until your program is terminated.

It was because I noticed this issue that I added another method of managing memory to Landmark. Just free it yourself. There's a function in Landmark called `dealloc` which does just that. This function HOWEVER only operates on reference types because they are the only types primitive to Landmark which can represent a contiguous allocation of memory.

The `dealloc` function would get the lifetime of the chunk of memory you wish to free from its runtime reference. After getting the lifetime, it removes the chunk of memory you are freeing from the said lifetime. It is worth noting though that the `dealloc` function's full signature looks something like `fn dealloc(mem: any)` which implies that you HAVE to cast the reference to an `any` type before passing it to the dealloc function.

<!--
# Getting Started with Landmark
<aside name="home">
If you are reading this, I might want to assume you technically do not have the development environment. 

If that is truly the case, you can download the development environment from [here](https://github.com/landmark-lang/.github).

</aside>
-->
