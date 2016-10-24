---
title: "How to implement a new DOM API for Servo"
layout: post
---

[Servo](https://github.com/servo/servo/) is an awesome project about making a web browser engine in Rust! It is really well maintained with friendly contributors and mentors. Because Servo is a new web browser engine, there are so many relatively easy tasks waiting for new contributors. One of the things you as a contributor can do is implementing a DOM API for Servo.

[Malisa](https://twitter.com/malisas7) and I implemented a few DOM APIs over the summer. There are lots of great documentation on how to get started, but we still felt a bit lost in the beginning. So we decided to make a tutorial for people who are relatively new to Servo and want a little more concrete example on implementing a new DOM API for Servo.

In this blog post, I will implement the up and coming DOM API called ✨**Doge**✨. This API has a spec that describes the standard Doge API should follow. [You should give it a very brief skim before continuing](https://jeenalee.github.io/doge-standard/).

Doge is a relatively simple DOM API. You can create a new Doge object that has an associated *such list*. If an initializer is given, that will be used to populate the new Doge. You can add more words to *such list* through the `append` method. You can get a random word from *such list* through the `random` method. Simple! So, how do we start?

## Architecture
These are the most relevant directories for us. `script` is a crate, which includes a module `dom`. `webidls` hosts all the Web IDL files for the DOM APIs:
{% highlight console %}
.
├── components
│   ├── script
│   |   ├── dom
│   |   |   ├── webidls
...
{% endhighlight %}

## Web IDL
First, we should get the Web IDL that is in the spec. Copy them, and put them in `componenets/script/dom/webidls/Doge.webidl`. For now, we'll comment out the methods for Doge in the Web IDL. It's easiest to comment them all out, and then implement one method at a time. It will look like below after adding the license and the spec link:
{% highlight console %}
/* This Source Code Form is subject to the terms of the Mozilla Public
 * License, v. 2.0. If a copy of the MPL was not distributed with this
 * file, You can obtain one at http://mozilla.org/MPL/2.0/. */

// https://jeenalee.github.io/doge-standard/

typedef sequence<DOMString> DogeInit;

[Constructor(optional DogeInit init),
 Exposed=(Window,Worker)]

interface Doge {
  // void append(DOMString word);
  // DOMString random();
};
{% endhighlight %}

If this is the first time adding a Web IDL to Servo from this specific domain, make sure to add it to `python/tidy/servo_tidy/tidy.py` so that `test-tidy` will know it's a valid Web IDL standard.

## Listing Doge as a submodule
`doge.rs` is a submodule under `dom` so we should make it discoverable by the rest of Servo. We do so by listing the submodule in the `lib.rs` or `mod.rs` at the root of the module. In our case, we would add the following line to `componenets/script/dom/mod.rs`:
{% highlight rust %}
pub mod doge;
{% endhighlight %}

## Implementing Doge
Doge is a dom struct and has an associated *such list*. So we can write that like the following in `components/script/dom/doge.rs`:
{% highlight rust %}
#[dom_struct]
pub struct Doge {
    reflector_: Reflector,
    such_list: DOMRefCell<Vec<DOMString>>,
}
{% endhighlight %}
Because everything we write will be in Rust but the web speaks JavaScript, we need something that can connect the JS world and the Rust world. This will be `reflector_`, which provides a pointer to the JS object. We want `such_list` to be able to be modified, so we'll allow interior mutability by wrapping it with `DOMRefCell`. `DOMRefCell` is like `RefCell`, but it enables JS garbage collector tracing.

Let's write the functions required for creating a new Doge object!

#### **impl Doge**
We need three functions for creating a new Doge object: `new_inherited`, `new` and `Constructor`. `Constructor` is called by the JS world, which calls `new`, which in turn calls `new_inherited`. Let's work on `new_inherited`, `new`, and then `Constructor`.

- `new_inherited`:
This is where an instance of the Doge struct is created. Sometimes, parameters may be used here, but Doge doesn't require any (because Doge is a strong independent doggo💕):
{% highlight rust %}
pub fn new_inherited() -> Doge {
    Doge {
        reflector_: Reflector::new(),
        such_list: DOMRefCell::new(vec![]),
    }
}
{% endhighlight %}
- `new`:
This is where a connection between the JS world (`global`) and the Rust Doge object is created. It does so by calling a function `reflect_dom_object`, which takes three parameters including one called `DogeWrap`. `DogeWrap` is a DOM bindings generated code (more on that later) that sets up the JS reflector (`reflector_`) for the Doge type. `new` method returns a JS Rooted object (`Root<Doge>`), which will prevent the JavaScript garbage collector from collecting this object while it's used in the Rust world.
{% highlight rust %}
pub fn new(global: GlobalRef) -> Root<Doge> {
        reflect_dom_object(box Doge::new_inherited(), global, DogeWrap)
}
{% endhighlight %}
- `Constructor`:
This function constructs the new Doge object, and will add some fields if an initializer (`init`) is given. In our case, Doge can be given a `DogeInit` that is a sequence of `DOMString`. The spec for the constructor method states that:

>
  1. Let *doge* be a new `Doge` object.
  2. If *init* is given, for each *word* in *init*, append *word* to the associated *such list*.
  3. Return *doge*.

So, we follow the spec, and write the constructor! `Constructor()` must return the `Fallible` type, which is one of Servo's custom `Result` types.
{% highlight rust %}
// https://jeenalee.github.io/doge-standard/#dom-doge
pub fn Constructor(global: GlobalRef, init: Option<DogeInit>) -> Fallible<Root<Doge>> {
    // Step 1
    let doge = Doge::new(global);
    // Step 2
    if let Some(i) = init {
        for word in i {
            doge.Append(word);
        }
    }
    // Step 3
    Ok(doge)
}
{% endhighlight %}

We can almost create a Doge object. This won't compile yet, because the append method is not implemented. Awesome! Let's start writing the Doge methods.

#### **impl DogeMethods for Doge**
**`append` method**

Let's uncomment the `append` method from the Web IDL.
{% highlight console %}
interface Doge {
  void append(DOMString word);
  // DOMString random();
};
{% endhighlight %}
Now we'll try to compile. This will definitely fail! *However*, the DOM bindings generator creates a code related to Doge. You can find that in `target/debug/build/script-[hash]/out/Bindings/DogeBinding.rs` or something like that. This is also where you can find `DogeWrap`, which I mentioned earlier. Search for `DogeMethods`, and it will show:
{% highlight rust %}
pub trait DogeMethods {
    fn Append(&self, word: DOMString) -> ();
}
{% endhighlight %}
Oh how neat! Now we know exactly how to start writing the `append` method.

The spec for append states that:

>
The `append(word)` method, when invoked, must append `word` to the associated *such list*.

So we know that we should write the method like the following:
{% highlight rust %}
// https://jeenalee.github.io/doge-standard/#dom-doge-append
fn Append(&self, word: DOMString) -> () {
    self.such_list.borrow_mut().push(word);
}
{% endhighlight %}
One down, one more to go!

**`random` method**

The spec states:

>
The `random()` method, when invoked, must run these steps:
>
1. Let *list* be the associated *such list*.
2. If *list* is empty, throw a `TypeError`.
3. Return a randomly chosen word from *list*.

Because `random` method can *throw* an error, we should specify it in our Web IDL. Add `[Throws]` in front of `random` like this:
{% highlight console %}
interface Doge {
  void append(DOMString word);
  [Throws] DOMString random();
};
{% endhighlight %}
Let's now write the `random` method per spec:
{% highlight rust %}
// https://jeenalee.github.io/doge-standard/#dom-doge-random
fn Random(&self) -> Fallible<DOMString> {
    // Step 1
    let list = self.such_list.borrow();
    // Step 2
    if list.len() == 0 {
        return Err(Error::Type("Such list is empty".to_string()));
    } else {
        // Step 3
        let random_index = rand::thread_rng().gen_range(0, list.len());
        return Ok(list[random_index].clone());
    }
}
{% endhighlight %}

That's it! We implemented the Doge API for Servo. Let's try it out!

[![doge demo](/pics/doge/doge-demo.gif)](/pics/doge/doge-demo.gif)

<p><font face="Comic Sans MS" color="#8e00df">Wow!</font> <font face="Comic Sans MS" color="#40df00">Such amazing!</font> <font face="Comic Sans MS" color="#0074df">Very cool.</font></p>

To see the full implementation, check out the [doge commit in my Servo fork](https://github.com/jeenalee/servo/commit/18e758655fead3b32cdcdc04c2b3dd21472153de).

This tutorial does not cover everything related to implementing a DOM API, but I hope this helped you get started! Sometimes the spec will be unclear, and sometimes it will be impossible to implement an API exactly as the spec states. When you encounter a situation like that or when you need help, reach out to Servo and spec maintainers! They are friendly and helpful people. 😊

Malisa wrote a [blog post about the DOM API testing process in Servo](http://hellomalisa.me/2016-10-24/how-to-test-a-servo-dom-api.html), so read about it!

<small>Many thanks to [Josh Matthews](https://twitter.com/lastontheboat) (`jdm`) for reviewing the blog post.</small>
