# General 
Don't think of this as a regular ol' README file. Think of it like a happy child's Journal. This is an informative publication about the Landmark programming language and my the process and design choices while I was building it. Also, far below — after all of my chaotic thoughts — you'd be reading here, you would find documentation containing some snippets of Landmark code and others containing how to get started with using Landmark. After that description you might think of this as a whitepaper and that's wrong for three reasons:

1. Meh, you can argue that there's barely a problem I am trying to solve that another language which exists already does not solve. Although, I would talk about a few problems and the way Landmark tries to solve or sometimes even solves them.
2. Err... the pilot version of the compiler is written in Golang. No matter what reason sophisticated enough that I try to come up with, the tool would defeat this reason.
3. It's kind of weird calling it whitepaper when basically anybody reading this would read it in dark mode.

# And then it churned
I wanted to write a back-end framework in C but I quickly realized that it was a task that's too complicated for people. There were too many things that I needed to do myself like security, parallelization, extensibility, templating, and so on. The more code I fleshed out for this framework, the greater the amount of emptiness that was left in my soul. Then, in a beautiful halo of light, Rust descended from the skies and seemed perfectly suitable to fix all of the problems I was facing.

When I started playing around with Rust, and experimenting with stripped down versions of the framework I envisioned, I quickly realized that it was too complicated for people. While working out the parallelization pipeline in Rust, I felt like my head was leaving my body trying to find my way around lifetimes, and Thread programming in Rust. Much like the time I tried building the framework in C, you might argue it was a skill issue. Well... who knows?

At that time, I made the most intelligent decision and gave up on building the framework altogether. So, right now, why am I currently making the foolish decision of building this framework you might ask? While working with Rust, I fell in love with its syntax, memory management model (this is a partial truth), type system, and architecture. Then I thought, how can I borrow some concepts from Rust while making it more friendly for beginners? It was in that moment that I either conceived Landmark or I f**ked up — only time will tell.

## Designing Landmark
There were a lot of things I wanted to borrow from Rust but I had to pick my favorites so I stay in scope. The properties of Landmark that were inspired by Rust are its type system, memory management strategy, syntax, and how it handles abstraction (one more half-truth but we would talk about this in a bit).

# Getting Started
<aside name="home">
If you are reading this, I might want to assume you technically do not have the development environment. 

If that is truly the case, you can download the development environment from [here](https://github.com/landmark-lang/.github).

</aside>
