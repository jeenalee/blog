---
title: "Moving, Cloning, and Copying Coloring Books in Rust"
layout: post
---
As a new Rustacean, I had heard about how awesome the Rust borrow system is, but I had not quite understood why. One thing I learned about myself recently is that if I wanted to be an efficient learner, it is important for me to understand the 'why' part.

So, that brings me to the question: why is Rust's borrow system considered great? Why do seasoned rustaceans strongly prefer borrowing over copying or cloning variables?

In order to understand the answers to those questions, I had to understand what's happening when a variable is moved, cloned, or copied. My background is genetics, not computer science, so to understand `move`, `copy`, and `clone`, I had to learn the basic anatomy of computer memory: [the stack and the heap](https://doc.rust-lang.org/book/the-stack-and-the-heap.html).

This blog post is a review/note to myself about these concepts in the context of Rust: what happens in the stack and the heap when a variable is moved, cloned or copied.

## The Stack and The Heap

Computer memory is like the human memory in the brain; it stores information. The heap and the stack are arbitrary regions in the memory with different functions, just like how we arbitrarily divided the human brain into different lobes. Like how the frontal lobe of the brain carries out functions that are different from occipital lobe, the stack and heap have different responsibilities.

The stack stores information about where in the heap to find more detailed information (unless it's a special kind of information which we'll cover later), like a library directory. It will tell us where to find the data we need, i.e. "The Harry Potter series starts at the shelf 1 and continues for 3 shelves". Just like how we wouldn't add the entire script of the Harry Potter book in the directory every time a new book came out, the stack doesn't contain the actual data (except for primitive types which we'll cover later).

The actual data would be stored in the heap. Because the heap reserves some extra room for the data, the data can grow or shrink. Think of library shelves that are about half empty so that the librarian can add more books if they need to. Without this extra empty space, the librarians will have to move the entire Harry Potter series every time JK Rolling wrote another book!

## Move
So what does the stack and the heap look like when I create a variable binding in Rust? Let's reuse the coloring book analogy. Say there is a coloring book named `free_coloring_book` in which each page is dedicated to a planet of our solar system! And it's up for grabs!

{% highlight rust %}
let free_coloring_book = vec!["mercury", "venus", "earth",
                              "mars", "jupiter", "saturn",
                              "uranus", "neptune"];
{% endhighlight %}

I used a [`vec`](https://doc.rust-lang.org/book/vectors.html) to represent the coloring book. The data in the vector (planets, in our example) lives in the heap, and the vector can grow if needs be. What does this look like in the memory? (not so accurately):]

{% highlight bash %}
length capacity pointer         heap
  8       16      ->            ... | "mercury" | ..(6 other planets).. | "neptune" | ...
{% endhighlight %}

The stack contains short information about the data on the heap. `length` shows how long the vector is. We have 8 planets, so that makes it `8`. `capacity` shows how much room is reserved for this vector in case it grows. It's usually about twice as big as `length`, hence `16`. `pointer` contains the address to the `heap`, where the actual data live.

Now, let's say a friend claims ownership to this `free_coloring_book`! Our friend will make it mutable, so that they can add or reduce pages as they want.

{% highlight rust %}
let mut friends_coloring_book = free_coloring_book;
{% endhighlight %}

What would this look like in the memory?

{% highlight bash %}
length capacity pointer         heap
  8       16      ->              ... | "mercury" | ..(6 other planets).. | "neptune" | ...
  8       16      (not usuable)
{% endhighlight %}

`free_coloring_book` is no longer accessible and the Rust compiler would tell us if we tried to access it.

{% highlight rust %}
println!("Free coloring book looks like:\n {:?}", free_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/cZIzFU) and the compiler's message:

{% highlight console %}
error: use of moved value: `free_coloring_book` [--explain E0382]
 --> <anon>:7:53
  |>
5 |>     let friends_coloring_book = free_coloring_book;
  |>         --------------------- value moved here
6 |>
7 |>     println!("Free coloring book looks like:\n {:?}", free_coloring_book);
  |>                                                       ^^^^^^^^^^^^^^^^^^ value used here after move
{% endhighlight %}

It's because the value of `free_coloring_book` *moved* to `friends_coloring_book`. And we can see that `friends_coloring_book` is totally accessible!:

{% highlight rust %}
println!("Our friend's coloring book looks like: {:?}", friends_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/SC86Xx) and its output:

{% highlight console %}
Our friend's coloring book looks like:
 ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune"]
{% endhighlight %}

Upon flipping through the pages of the coloring book, our friend gets disappointed that the coloring book missed out on pluto. So they decide to add pluto to their coloring book. Because the coloring book is a vector, they can add a page pretty easily:

{% highlight rust %}
friends_coloring_book.push("pluto");
println!("Our friend's coloring book after adding pluto: \n {:?}", friends_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/39wUoj) and its output:

{% highlight console %}
Our friend's coloring book after adding pluto:
 ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune", "pluto"]
{% endhighlight %}

What would happen in the memory?

{% highlight console %}
length capacity pointer         heap
  9       16      ->              ... | "mercury" | ..(6 other planets).. | "neptune" | "pluto" | ...
  9??     16      (not usuable)
{% endhighlight %}

`capacity` remains the same, because the vector still has enough room in case it grows. `length`, however, is now changed to 9. This allows the pointer to know that the length of the vector is 9, starting at location #.

## Clone
So far, our friend claimed ownership of the `free_coloring_book`, and they modified it by adding `"pluto"` to the book. What if you didn't like this change? You decide to make a *clone* of our friends coloring book so that you can make your own version:

{% highlight rust %}
let mut my_coloring_book = friends_coloring_book.clone();
{% endhighlight %}

{% highlight console %}
length capacity pointer         heap
  9       16      ->              ... | "mercury" | ..(6 other planets).. | "neptune" | "pluto" | ...
  9       16      ->              ... | "mercury" | ..(6 other planets).. | "neptune" | "pluto" | ...
  9??     16      (not usuable)
{% endhighlight %}

When the data is cloned, it creates an exactly identical copy of the data that is independent of the original data. You argue that pluto is a dwarf planet, and should not be on a coloring book of planets. So you remove the last element (`"pluto"`) from `my_coloring_book`. What happens?

{% highlight rust %}
my_coloring_book.pop();

println!("My coloring book after removing pluto:\n {:?}", my_coloring_book);
println!("Our friend's coloring book:\n {:?}", friends_coloring_book);
{% endhighlight %}

[Link to code](https://is.gd/YKtK4c) and its output:

{% highlight console %}
My coloring book after removing pluto:
 ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune"]
Our friend's coloring book:
 ["mercury", "venus", "earth", "mars", "jupiter", "saturn", "uranus", "neptune", "pluto"]
{% endhighlight %}

`my_coloring_book` and `friends_coloring_book` are different! They are independent of each other, and that's why they are both accessible by `println!`.

{% highlight console %}
length capacity pointer         heap
  8       16      ->              ... | "mercury" | ..(6 other planets).. | "neptune" | "pluto" | ...
  9       16      ->              ... | "mercury" | ..(6 other planets).. | "neptune" | "pluto" | ...
  9??     16      (not usuable)
{% endhighlight %}

Now, the data in the heap still includes `"pluto"`, but the length pointer on the stack has decreased by one. This way, the computer can be lazy about actually truncating data, which would be computationally costly! Clever!

## Copy
Things work slightly different with [primitive types](https://doc.rust-lang.org/book/primitive-types.html). When the ownership of a primitive type is "moved" to a different binding, a separate *copy* actually gets created, and the ownership isn't moved! This is because primitive types (which include numeric values and string) implement a `Copy` trait, and their actual data live in the stack! Let's see what this means. In my [previous blog post](http://localhost:4000/2016/08/15/sharing-coloring-books-in-rust.html), I wrote:

{% highlight rust %}
fn main() {
    let mut x = 5;
    let mut y = x;

    y += 1;

    println!("x is: {}", x);
    println!("y is: {}", y);
}
{% endhighlight %}

The reason the compiler was able to print both `x` and `y` is that `x` and `y` are different. Unlike with vectors in which case the value moved, when the compiler sees `let mut y = x;`, it creates `y` as a separate copy of `x`. What does this look like in the stack? Let's visualize it:

{% highlight console %}
  value    name
    5        y
    5        x
{% endhighlight %}

And when `y` increments by `1`:

{% highlight console %}
  value    name
    6        y
    5        x
{% endhighlight %}

When we create a reference, like we did in the previous blog post:

{% highlight rust %}
let mut x = 5;
let mut y = &x;
{% endhighlight %}

{% highlight console %}
  value    name
    <-       y
    5        x
{% endhighlight %}

So when you make a change like `y += 1`, it would look like:

{% highlight console %}
  value    name
    <-       y
    6        x
{% endhighlight %}

Interesting! By incrementing `y`, `x`'s data was in fact modified.

Let's go back to the question I started with: why do seasoned rustaceans strongly prefer borrowing over cloning or copying variables? It's because by borrowing instead of cloning or copying, you can save potentially a huge amount of memory space!

Also, every piece of data has an owner, and when the owner goes out of scope, the memory bits that were hosting the data are freed as well. This removes the need of garbage collection, which is a computationally expensive process! This makes Rust super fast, and is another reason Rust's borrow system is awesome.

I've been really enjoying learning Rust, because it has encouraged me to think about what's happening *inside* the computer when I type something. By copying and cloning, is my program taking up more than necessary space in the memory? Can I make a reference instead? What is happening when I add or delete something? So much to learn!
