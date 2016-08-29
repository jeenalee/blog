---
title: "Moving, Cloning, and Copying Coloring Books in Rust"
layout: post
---

As a new Rustacean, I had heard about how awesome the Rust borrow
system is, but I had not quite understood why. One thing I learned
about myself recently is that if I wanted to be an efficient learner,
it is important for me to understand the 'why' part.

So, that brings me to the question: why is Rust's borrow system
considered great? Why do seasoned Rustaceans strongly prefer borrowing
over copying or cloning variables?

In order to understand the answers to those questions, I had to
understand what's happening when a variable is moved, cloned, or
copied. My background is genetics, not computer science, so to
understand `move`, `copy`, and `clone`, I had to learn the basic
anatomy of computer memory:
[the stack and the heap](https://doc.rust-lang.org/book/the-stack-and-the-heap.html).

This blog post is a review/note to myself about these concepts in the
context of Rust: what happens in the stack and the heap when a
variable is moved, cloned or copied.

## <a href="#the_stack_and_the_heap" id="the_stack_and_the_heap">The Stack and The Heap</a>

Computer memory is like the human memory in the brain; it stores
information. The heap and the stack are separate regions in the memory
with different functions like how different parts of the human brain
have different responsibilities.

The stack is like, as [Liene](https://twitter.com/li3n3) described,
[Pringles](https://en.wikipedia.org/wiki/Pringles)! Instead of a stack
of potato chips, the computer's stack is a stack of information! Like
how there is a limited amount of space in a Pringles can, the stack
has a limited amount of space in memory. The stack also has a set of
strict rules:

1. You can only remove data at the top of the stack. Can you pull out
   the chip in the middle of the Pringles can without removing all the
   chips above it? No.

2. Similarly, you can only add a new piece of data to the top of the
   stack. With Pringles, how do you add a new chip to the middle of
   the can without altering the whole can of chips? You can't. This
   also means that the age of the data on the top of the stack is
   younger than the data on the bottom.

Any piece of data that will be stored in the stack must know its size
at compile time. I can store an array in the stack if I know that it
has a static length, and will not grow or shrink. What if I didn't
know how long my data will be at runtime?  If I were to store it in
the stack and tried to increase its length, it will violate the stack
rules because I can't insert new data in the middle of the stack. That is
why we created the heap.

The heap is like a large space, where any data can claim a free spot
as long as it fits. For example, I can have a vector on the stack that
has an address pointing to the heap, where its elements reside. That
way, I can grow or shrink the number of elements in the vector as much
as I want without violating the stack rules.

There is one cost to the heap's flexibility: extra clean up. With the
stack, data is cleaned up as it goes out of scope. With the heap, we
need to clean it up ourselves. For example, when a vector is no longer
in use, it must take care to also destroy its elements residing in the
heap.

## <a href="#move" id="move">Move</a>

So what does the stack and the heap look like when I create a variable
binding in Rust? Say there is a coloring book named
`free_coloring_book` where each page is dedicated to a planet of our
solar system! And it's up for grabs!

{% highlight rust %}
let free_coloring_book = vec![
    "mercury",
    "venus",
    "earth",
    "mars",
    "jupiter",
    "saturn",
    "uranus",
    "neptune"
];
{% endhighlight %}

I used a [`vec`](https://doc.rust-lang.org/book/vectors.html) to
represent the coloring book. `free_coloring_book` is a vector of
`&str` string literals. The data in the vector (references to our
planet strings, in our example) lives in the heap, and the vector can
grow if needs be. What does this look like in the memory?

![freecoloringbook](/pics/moveclonecopy/freecoloringbook.jpg)

The stack contains metadata about the data on the
heap. `free_coloring_book` is a vector that owns 8 elements, and
therefore its `length` is `8`. `capacity` shows how much room is
reserved for this vector in case it grows. In this case, it's about
twice as big as `length`, hence `16`. `pointer` contains the address
in the `heap`, where the actual elements live. Each element is a pair
of a pointer to a `str` and its length, known as a *slice* in Rust's
parlance. For example, `"mercury"` has a length of 7.

Now, let's say a friend claims ownership to this `free_coloring_book`!
Our friend will make it mutable, so that they can add pages if they
want to.

{% highlight rust %}
let mut friends_coloring_book = free_coloring_book;
{% endhighlight %}

What would this look like in the memory?

![move1](/pics/moveclonecopy/move1.jpg)

`free_coloring_book` is no longer accessible, and the Rust's borrow
checker will tell us if we try to access it.

{% highlight rust %}
println!("Free coloring book looks like:\n {:?}", free_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/cZIzFU) and the compiler's message:

{% highlight console %}
error: use of moved value: `free_coloring_book` [--explain E0382]
 --> <anon>:7:55
  |>
5 |>     let mut friends_coloring_book = free_coloring_book;
  |>         ------------------------- value moved here
6 |>
7 |>     println!("Free coloring book looks like:\n {:?}", free_coloring_book);
  |>                                                       ^^^^^^^^^^^^^^^^^^ value used here after move
<std macros>:2:27: 2:58: note: in this expansion of format_args!
<std macros>:3:1: 3:54: note: in this expansion of print! (defined in <std macros>)
<anon>:7:5: 7:75: note: in this expansion of println! (defined in <std macros>)
note: move occurs because `free_coloring_book` has type `std::vec::Vec<&str>`, which does not implement the `Copy` trait
{% endhighlight %}

It's because the value of `free_coloring_book` *moved* to
`friends_coloring_book`. And we can see that `friends_coloring_book`
is totally accessible!:

{% highlight rust %}
println!("Our friend's coloring book looks like: {:?}", friends_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/SC86Xx) and its output:

{% highlight console %}
Our friend's coloring book looks like: ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune"]
{% endhighlight %}

Upon flipping through the pages of the coloring book, our friend gets
disappointed that the coloring book missed out on Pluto. So they
decide to add Pluto to their coloring book. Because the coloring book
is a vector, they can add a page pretty easily:

{% highlight rust %}
friends_coloring_book.push("pluto");
println!("Our friend's coloring book after adding Pluto: \n {:?}", friends_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/J470Yw) and its output:

{% highlight console %}
Our friend's coloring book after adding Pluto: ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune", "pluto"]
{% endhighlight %}

What would happen in the memory?

![after adding pluto](/pics/moveclonecopy/move2.jpg)

`capacity` remains the same, because the vector still has enough room
in case it grows. `length`, however, is now changed to 9. This allows
the pointer to know that the length of the vector is 9, starting at
location `0xf00`.

## <a href="#clone" id="clone">Clone</a> ##

So far, our friend claimed ownership of the `free_coloring_book`, and
they modified it by adding `"pluto"` to the book. What if you didn't
like this change? You decide to make a *clone* of our friends coloring
book so that you can make your own version:

{% highlight rust %}
let mut my_coloring_book = friends_coloring_book.clone();
{% endhighlight %}

![after cloning](/pics/moveclonecopy/clone1.jpg)

When the data is cloned, it creates an exactly identical copy of the
data that is independent of the original data. You argue that Pluto is
a dwarf planet, and should not be on a coloring book of planets. So
you remove the last element (`"pluto"`) from `my_coloring_book`. What
happens?

{% highlight rust %}
my_coloring_book.pop();

println!("My coloring book after removing Pluto:\n {:?}", my_coloring_book);
println!("Our friend's coloring book:\n {:?}", friends_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/vh9Cdm) and its output:

{% highlight console %}
My coloring book after removing Pluto: ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune"]
Our friend's coloring book: ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune", "pluto"]
{% endhighlight %}

`my_coloring_book` and `friends_coloring_book` are different! They are
independent of each other, and that's why they are both accessible by
`println!`. How would our coloring books look like in the memory?

![after popping pluto](/pics/moveclonecopy/clone2.jpg)

Now, the data in the heap still includes `"pluto"`, but the length
pointer on the stack has decreased by one. This way, the computer can
be lazy about actually truncating data, which would be computationally
costly. Clever!

## <a href="#copy" id="copy">Copy</a>

Things work slightly different with types that don't require storing
descendent data in the heap, such as a number or a character. These
special types don't own other elements on the heap like a vector
does. These types implement the `Copy` trait, which means when the
data is duplicated, it is copied bit-by-bit at the surface level. If
this were to happen with a vector, the copied object would result in a
duplicate pointer to the heap, and it would not be clear which copy is
responsible for destroying the heap elements. But, a surface level
copy is sufficient for `Copy` types because their data is simple and
flat.

Let's see what this means. In my
[previous blog post](/2016/08/15/sharing-coloring-books-in-rust.html),
I wrote:

{% highlight rust %}
fn main() {
    let mut x = 5;
    let mut y = x;

    y += 1;

    println!("x is: {}", x);
    println!("y is: {}", y);
}
{% endhighlight %}

The reason the compiler was able to print both `x` and `y` is that `x`
and `y` are different values on the stack. Unlike with vectors in
which case the value moved, when the compiler sees `let mut y = x;`,
it creates `y` as a separate copy of `x` because `x` is a `Copy`
type.

What does this look like in the stack? Let's visualize it:

<img src="/pics/moveclonecopy/copy1.jpg" alt="when y is moved, x is copied" style="width: 200px;"/>

A number `5` doesn't own other data types, so `x` and `y` are able to
be stored only in the stack. When `y` increments by `1`:

<img src="/pics/moveclonecopy/copy2.jpg" alt="y is a separate thing
from x" style="width: 200px;"/>

When we create a reference, like we did in the previous blog post:

{% highlight rust %}
let mut x = 5;
let mut y = &x;
{% endhighlight %}

<img src="/pics/moveclonecopy/copy3.jpg" alt="when y is a reference to
x, it has a pointer" style="width: 200px;"/>

So when you make a change like `*y += 1`, it would look like:

<img src="/pics/moveclonecopy/copy4.jpg" alt="when y is a reference to
x and make changes to y, it's applied to x" style="width: 200px;"/>

Interesting! By incrementing `*y`, `x`'s data was modified.

Let's go back to the question I started with: why do seasoned
Rustaceans strongly prefer borrowing over cloning or copying
variables? It's because by borrowing instead of cloning or copying,
you can increase performance and potentially save huge amounts of
memory!

Also, if you follow data ownership all the way back to the the owning
variable, when that variable goes out of scope, the whole tree of
owned data is cleaned up. This removes the need for garbage
collection, which can be a computationally expensive process. This
makes Rust super fast, and is another reason Rust's borrow system is
awesome.

I've been really enjoying learning Rust, because it has encouraged me
to think about what's happening *inside* the computer when I type
something. By copying and cloning, is my program taking up more than
necessary space in the memory? Can I make a reference instead? How is
memory managed differently in other languages, like Python?

So much to learn!
